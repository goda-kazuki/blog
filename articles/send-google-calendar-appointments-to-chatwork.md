---
title: "Googleカレンダーで組織内の予定をChatWorkにGASを使って通知してみた"
published_at: "2021-11-20"
type: "tech"
published: true
coverImage: "google-calendar-1.png"
---

## はじめに

うちの会社ではGoogle Workspace(旧 Gsuite)を契約しており、それぞれの予定はGoogleカレンダーに登録しています。 全体朝礼で、その日の予定を報告するのですが人数が多く聞き取れなかったりするので、不便に感じていました。

Googleカレンダーのページで全員を追加すると、1日に何件もの予定が表示されてしまい、とても使える状態ではありませんでした。 参加者が複数の場合、それぞれに対して予定が表示されるのも原因に感じます。

![全員表示の時のGoogleカレンダー](/images/全員表示の時のGoogleカレンダー-1024x524.jpg)

そこで、組織内全員の予定を一括で取得し、まとめた形で出力することで上記の不便を解消しました。

## 前提

私はGoogle Workspaceで管理者の権限を持っていません。 なので、今回の内容は組織内のユーザであれば、基本的に誰でも実装することができます。

また、とりあえず動けばいいやで作ったので、コードが汚いです。ご了承ください・・。

## どんなことができたのか

毎日、以下のようなメッセージがChatWorkに送られるようになりました。 これで、個々が報告する必要が無くなったのと、記録することができるようになりました。

![ChatWorkへの送信結果](/images/ChatWorkへの送信結果.png)

## やり方

googleのcalendar apiとadmin-sdkを使用しています。

お金をかけたくないので、GAS(Google Apps Script)を使用して実装しました。 コードは全てこちらにあります。 [https://github.com/goda-kazuki/send-google-Calendar-appointments-to-ChatWork/blob/main/code.js](https://github.com/goda-kazuki/send-google-Calendar-appointments-to-ChatWork/blob/main/code.js)

順番としては、以下の通りです。メインの部分しか記載しませんので、適宜[Github](https://github.com/goda-kazuki/send-google-Calendar-appointments-to-ChatWork/blob/main/code.js)を参照ください。

1. 組織に属しているユーザのリストを取得
2. 取得したユーザの予定を取得
3. 予定をまとめ、ChatWorkに送信する

### 組織に属しているユーザのリストを取得

社員それぞれのメールアドレスが一意なので、メールアドレスを全員分取得します。

```javascript
// 参考 https://developers.google.com/apps-script/advanced/admin-sdk-directory
function listAllUsers() {
  let page;
  const result = [];
  // https://developers.google.com/admin-sdk/directory/reference/rest/v1/users/list
  page = AdminDirectory.Users.list({
    domain: '{{組織のドメイン名 ◯◯.comなど}}',
    viewType: 'domain_public',
  });
  const users = page.users;
  for (let i = 0; i < users.length; i++) {
    const user = users[i];
    result.push({ name: user.name.fullName, email: user.primaryEmail });
  }
  pageToken = page.nextPageToken;
  return result;
}
```

### 取得したユーザの予定を取得

1で取得したユーザー情報を元に、予定を取得していきます。

```javascript
function getEventListFromUsers(users) {
  const timezone = Session.getScriptTimeZone();
  const RFC3339format = 'yyyy-MM-dd\'T\'HH:mm:ss.SXXX';

  const now = new Date();
  now.setHours(0, 0, 0, 0);
  let date = Utilities.formatDate(now, timezone, RFC3339format);
  const timeMin = Utilities.formatString('%s', date);

  now.setHours(23, 59, 59, 59);
  date = Utilities.formatDate(now, timezone, RFC3339format);
  const timeMax = Utilities.formatString('%s', date);

  const eventsFromUsers = [];

  for (let i = 0; i < users.length; i++) {
    const user = users[i];

    // https://developers.google.com/calendar/api/v3/reference/events/list
    const eventsFromUser = Calendar.Events.list(user.email, {
      timeMin: now.toISOString(),
      singleEvents: true,
      orderBy: 'startTime',
      timeMin: timeMin,
      timeMax: timeMax, 
    });

    for (let ii = 0; ii < eventsFromUser.items.length; ii++) {
      let event = eventsFromUser.items[ii];
      let eventInfo = {};
      eventInfo.title = event.summary;
      eventInfo.creator = findUserFullName(users, event.creator.email, '');
      eventInfo.user = user.email;

      // 時間指定のイベントの場合、開始時間を格納
      if (event.start.dateTime) {
        const eventStartDate = new Date(event.start.dateTime);
        eventInfo.eventStartDate = eventStartDate.toLocaleTimeString('en-US', { hour12: false });
        const eventEndDate = new Date(event.end.dateTime);
        eventInfo.eventEndDate = eventEndDate.toLocaleTimeString('en-US', { hour12: false });
      }

      // イベントに他の参加者がいる場合
      if (event.attendees) {
        let attendees = [];
        let resourceName = null;
        for (let iii = 0; iii < event.attendees.length; iii++) {
          const attende = event.attendees[iii];

          // リソース(会議室とか)の場合
          if (attende.resource) {
            resourceName = attende.displayName;
            continue;
          }
          attendees.push(findUserFullName(users, attende.email, attende.displayName));
        }

        eventInfo.attendees = attendees;
        if (resourceName !== null) {
          eventInfo.resourceName = resourceName;
        }
      }
      eventsFromUsers.push(eventInfo);

    }
  }

  const filteredUniqueEvents = filterUnique(eventsFromUsers);
  return filteredUniqueEvents.sort(alphabetically(true));
}
```

### 予定をまとめ、ChatWorkに送信する

ChatWorkに送信するためのライブラリを提供している方がいたので、そのライブラリを使用して送信しています。

ライブラリの使い方はこちら [https://c-note.chatwork.com/post/69590585422/chatworkapi-gas-library](https://c-note.chatwork.com/post/69590585422/chatworkapi-gas-library) [https://qiita.com/y\_minowa/items/39db8ca5bffc9c440caf](https://qiita.com/y_minowa/items/39db8ca5bffc9c440caf)

ちょっとゴリ押し感強いのですが、とりあえず送れます。

```javascript
//参考 https://developers.google.com/apps-script/advanced/calendar?authuser=0#listing_events
function sendMessageToChatWork() {
  const users = listAllUsers();
  const events = getEventListFromUsers(users);
  let sendText = '';

  let allDayScheduleText = '[info][title]終日予定[/title]';

  for (var i = 0; i < events.length; i++) {
    const event = events[i];

    if (! event.eventStartDate) {
      allDayScheduleText = allDayScheduleText + 'タイトル：' + event.title + '　作成者：' + event.creator + '\n';
      continue;
    }

    sendText = sendText + '[info][title]';
    sendText = sendText + '時間：' + event.eventStartDate + '〜' + event.eventEndDate;

    sendText = sendText + '　タイトル：' + event.title;

    if (event.resourceName) {
      sendText = sendText + '　会議室：' + event.resourceName;
    }

    sendText = sendText + '[/title]';
    if (event.attendees) {
      sendText = sendText + '参加者：' + event.attendees;
    } else {
      sendText = sendText + '参加者：' + event.creator;
    }
    sendText = sendText + '[/info]';
  }

  if(allDayScheduleText==='[info][title]終日予定[/title]'){
    allDayScheduleText='';
  }else{
    allDayScheduleText=allDayScheduleText + '[/info]';
  }

  sendText = sendText + allDayScheduleText;

  const client = ChatWorkClient.factory({ token: 'c4c571858b7c40896b67487c1612152a' });
  client.sendMessage({ room_id: '96421512', body: sendText });
}
```

## その他

ちょっとコード量が増えてきた時は、claspを使ったほうが開発しやすいと思います。 claspについては、過去記事参照いただけると嬉しいです。 [https://gdtypk.com/2019-11-21-000000/](https://gdtypk.com/2019-11-21-000000/)

## まとめ

管理者権限が無くても、GoogleのAPIをある程度使えるみたいで、勉強になりました。 他のことにも使えそうですね。
