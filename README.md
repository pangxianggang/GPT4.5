<!DOCTYPE html>

<html lang="zh-CN">

<head>

<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>DeepSeek 聊天机器人</title>

<style>

body {

font-family: Arial, sans-serif;

background-color: #f0f8ff;

display: flex;

justify-content: center;

align-items: center;

height: 100vh;

margin: 0;

}

.chat-container {

background-color: #ffffff;

border: 1px solid #b0c4de;

border-radius: 8px;

box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);

width: 90%;

max-width: 600px;

padding: 20px;

display: flex;

flex-direction: column;

align-items: center;

height: 100%;

}

.chat-header {

text-align: center;

margin-bottom: 20px;

width: 100%;

background-color: #00008b;

color: #ffffff;

padding: 10px;

border-radius: 8px 8px 0 0;

}

.chat-header h1 {

margin: 0;

}

.input-field {

width: 100%;

display: flex;

flex-direction: column;

align-items: flex-start;

}

.input-field label {

margin-bottom: 5px;

color: #00008b;

}

.input-field input, .input-field textarea {

width: 100%;

padding: 10px;

border: 1px solid #b0c4de;

border-radius: 4px;

box-sizing: border-box;

}

.chat-box {

width: 100%;

flex-grow: 1;

max-height: 400px;

overflow-y: auto;

border: 1px solid #b0c4de;

border-radius: 4px;

padding: 10px;

background-color: #e0f7fa;

margin-bottom: 20px;

display: flex;

flex-direction: column;

scrollbar-width: thin;

scrollbar-color: #b0c4de #e0f7fa;

}

.chat-box::-webkit-scrollbar {

width: 8px;

}

.chat-box::-webkit-scrollbar-track {

background: #e0f7fa;

}

.chat-box::-webkit-scrollbar-thumb {

background-color: #b0c4de;

border-radius: 4px;

}

.message {

margin: 5px 0;

padding: 10px;

border-radius: 4px;

background-color: #ffffff;

display: flex;

justify-content: space-between;

align-items: center;

max-width: 90%;

word-wrap: break-word;

}

.message.user {

background-color: #add8e6;

align-self: flex-end;

}

.message.bot {

background-color: #e0f7fa;

align-self: flex-start;

}

.message .text {

flex-grow: 1;

word-wrap: break-word;

}

.message button {

background-color: #00008b;

color: #ffffff;

border: none;

border-radius: 4px;

cursor: pointer;

font-size: 12px;

padding: 5px 10px;

margin-left: 10px;

}

.message button:hover {

background-color: #0000cd;

}

.submit-button {

width: 100%;

padding: 10px;

background-color: #00008b;

color: #ffffff;

border: none;

border-radius: 4px;

cursor: pointer;

font-size: 16px;

}

.submit-button:hover {

background-color: #0000cd;

}

.clear-button {

width: 100%;

padding: 10px;

background-color: #ff4500;

color: #ffffff;

border: none;

border-radius: 4px;

cursor: pointer;

font-size: 16px;

margin-bottom: 10px;

}

.clear-button:hover {

background-color: #ff6347;

}

.prompt-field {

margin-bottom: 10px;

width: 100%;

}

.prompt-field label {

margin-bottom: 5px;

color: #00008b;

}

.prompt-field input {

width: 100%;

padding: 10px;

border: 1px solid #b0c4de;

border-radius: 4px;

box-sizing: border-box;

}

</style>

</head>

<body>

<div class="chat-container">

<div class="chat-header">

<h1>DeepSeek 聊天机器人</h1>

</div>

<button class="clear-button" onclick="clearChat()">清除对话</button>

<div class="prompt-field">

<label for="prompt">提示词:</label>

<input type="text" id="prompt" placeholder="请输入提示词">

</div>

<div class="input-field">

<label for="api-key">API 密钥:</label>

<input type="text" id="api-key" placeholder="请输入您的 API 密钥">

</div>

<div class="chat-box" id="chat-box">

<!-- 聊天内容将显示在这里 -->

</div>

<div class="input-field">

<label for="user-query">查询:</label>

<textarea id="user-query" rows="4" placeholder="请输入您的查询"></textarea>

</div>

<button class="submit-button" onclick="sendMessage()">发送</button>

</div>



<script>

async function sendMessage() {

const apiKey = document.getElementById('api-key').value;

const userQuery = document.getElementById('user-query').value;

const chatBox = document.getElementById('chat-box');

const prompt = document.getElementById('prompt').value;



if (!apiKey || !userQuery) {

chatBox.innerHTML += '<div class="message user"><div class="text">请确保输入您的 API 密钥和查询内容。</div> <button onclick="copyText(\'请确保输入您的 API 密钥和查询内容。\')">复制</button></div>';

return;

}



// 显示用户消息

chatBox.innerHTML += `<div class="message user"><div class="text">${userQuery}</div> <button onclick="copyText('${userQuery}')">复制</button></div>`;

document.getElementById('user-query').value = '';



// 显示正在处理的消息

const processingMessage = document.createElement('div');

processingMessage.className = 'message bot';


processingMessage.innerHTML = '<div class="text">正在处理您的请求...</div> <button onclick="copyText(\'正在处理您的请求...\')">复制</button>';

chatBox.appendChild(processingMessage);



try {

const response = await fetch('https://api.deepseek.com/chat/completions', {

method: 'POST',

headers: {

'Content-Type': 'application/JSON',

'Authorization': `Bearer ${apiKey}`

},

body: JSON.stringify({

model: "deepseek-chat",

messages: [

{"role": "system", "content": prompt || "您是一个乐于助人的助手"},

{"role": "user", "content": userQuery}

],

stream: false

})

});



if (!response.ok) {

throw new Error('网络响应不正常');

}



const data = await response.json();

const botResponse = data.choices[0].message.content;



// 更新正在处理的消息为实际回复


processingMessage.innerHTML = '<div class="text"></div> <button onclick="copyText(\'' + botResponse + '\')">复制</button>';

const textElement = processingMessage.querySelector('.text');

for (let i = 0; i < botResponse.length; i++) {

await new Promise(resolve => setTimeout(resolve, 50));

textElement.innerHTML += botResponse[i];

}

} catch (error) {


processingMessage.innerHTML = `<div class="text">发生错误: ${error.message}</div> <button onclick="copyText(\'发生错误: ${error.message}\')">复制</button>`;

} finally {

// 自动滚动到最新消息

chatBox.scrollTop = chatBox.scrollHeight;

}

}



function clearChat() {

const chatBox = document.getElementById('chat-box');

chatBox.innerHTML = '';

}



function copyText(text) {

navigator.clipboard.writeText(text).then(() => {

alert('文本已复制到剪贴板');

}).catch(err => {

console.error('无法复制文本: ', err);

});

}

</script>

</body>

</html><!DOCTYPE html>

<html lang="zh-CN">

<head>

<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>DeepSeek 聊天机器人</title>

<style>

body {

font-family: Arial, sans-serif;

background-color: #f0f8ff;

display: flex;

justify-content: center;

align-items: center;

height: 100vh;

margin: 0;

}

.chat-container {

background-color: #ffffff;

border: 1px solid #b0c4de;

border-radius: 8px;

box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);

width: 90%;

max-width: 600px;

padding: 20px;

display: flex;

flex-direction: column;

align-items: center;

height: 100%;

}

.chat-header {

text-align: center;

margin-bottom: 20px;

width: 100%;

background-color: #00008b;

color: #ffffff;

padding: 10px;

border-radius: 8px 8px 0 0;

}

.chat-header h1 {

margin: 0;

}

.input-field {

width: 100%;

display: flex;

flex-direction: column;

align-items: flex-start;

}

.input-field label {

margin-bottom: 5px;

color: #00008b;

}

.input-field input, .input-field textarea {

width: 100%;

padding: 10px;

border: 1px solid #b0c4de;

border-radius: 4px;

box-sizing: border-box;

}

.chat-box {

width: 100%;

flex-grow: 1;

max-height: 400px;

overflow-y: auto;

border: 1px solid #b0c4de;

border-radius: 4px;

padding: 10px;

background-color: #e0f7fa;

margin-bottom: 20px;

display: flex;

flex-direction: column;

scrollbar-width: thin;

scrollbar-color: #b0c4de #e0f7fa;

}

.chat-box::-webkit-scrollbar {

width: 8px;

}

.chat-box::-webkit-scrollbar-track {

background: #e0f7fa;

}

.chat-box::-webkit-scrollbar-thumb {

background-color: #b0c4de;

border-radius: 4px;

}

.message {

margin: 5px 0;

padding: 10px;

border-radius: 4px;

background-color: #ffffff;

display: flex;

justify-content: space-between;

align-items: center;

max-width: 90%;

word-wrap: break-word;

}

.message.user {

background-color: #add8e6;

align-self: flex-end;

}

.message.bot {

background-color: #e0f7fa;

align-self: flex-start;

}

.message .text {

flex-grow: 1;

word-wrap: break-word;

}

.message button {

background-color: #00008b;

color: #ffffff;

border: none;

border-radius: 4px;

cursor: pointer;

font-size: 12px;

padding: 5px 10px;

margin-left: 10px;

}

.message button:hover {

background-color: #0000cd;

}

.submit-button {

width: 100%;

padding: 10px;

background-color: #00008b;

color: #ffffff;

border: none;

border-radius: 4px;

cursor: pointer;

font-size: 16px;

}

.submit-button:hover {

background-color: #0000cd;

}

.clear-button {

width: 100%;

padding: 10px;

background-color: #ff4500;

color: #ffffff;

border: none;

border-radius: 4px;

cursor: pointer;

font-size: 16px;

margin-bottom: 10px;

}

.clear-button:hover {

background-color: #ff6347;

}

.prompt-field {

margin-bottom: 10px;

width: 100%;

}

.prompt-field label {

margin-bottom: 5px;

color: #00008b;

}

.prompt-field input {

width: 100%;

padding: 10px;

border: 1px solid #b0c4de;

border-radius: 4px;

box-sizing: border-box;

}

</style>

</head>

<body>

<div class="chat-container">

<div class="chat-header">

<h1>DeepSeek 聊天机器人</h1>

</div>

<button class="clear-button" onclick="clearChat()">清除对话</button>

<div class="prompt-field">

<label for="prompt">提示词:</label>

<input type="text" id="prompt" placeholder="请输入提示词">

</div>

<div class="input-field">

<label for="api-key">API 密钥:</label>

<input type="text" id="api-key" placeholder="请输入您的 API 密钥">

</div>

<div class="chat-box" id="chat-box">

<!-- 聊天内容将显示在这里 -->

</div>

<div class="input-field">

<label for="user-query">查询:</label>

<textarea id="user-query" rows="4" placeholder="请输入您的查询"></textarea>

</div>

<button class="submit-button" onclick="sendMessage()">发送</button>

</div>



<script>

async function sendMessage() {

const apiKey = document.getElementById('api-key').value;

const userQuery = document.getElementById('user-query').value;

const chatBox = document.getElementById('chat-box');

const prompt = document.getElementById('prompt').value;



if (!apiKey || !userQuery) {

chatBox.innerHTML += '<div class="message user"><div class="text">请确保输入您的 API 密钥和查询内容。</div> <button onclick="copyText(\'请确保输入您的 API 密钥和查询内容。\')">复制</button></div>';

return;

}



// 显示用户消息

chatBox.innerHTML += `<div class="message user"><div class="text">${userQuery}</div> <button onclick="copyText('${userQuery}')">复制</button></div>`;

document.getElementById('user-query').value = '';



// 显示正在处理的消息

const processingMessage = document.createElement('div');

processingMessage.className = 'message bot';


processingMessage.innerHTML = '<div class="text">正在处理您的请求...</div> <button onclick="copyText(\'正在处理您的请求...\')">复制</button>';

chatBox.appendChild(processingMessage);



try {

const response = await fetch('https://api.deepseek.com/chat/completions', {

method: 'POST',

headers: {

'Content-Type': 'application/JSON',

'Authorization': `Bearer ${apiKey}`

},

body: JSON.stringify({

model: "deepseek-chat",

messages: [

{"role": "system", "content": prompt || "您是一个乐于助人的助手"},

{"role": "user", "content": userQuery}

],

stream: false

})

});



if (!response.ok) {

throw new Error('网络响应不正常');

}



const data = await response.json();

const botResponse = data.choices[0].message.content;



// 更新正在处理的消息为实际回复


processingMessage.innerHTML = '<div class="text"></div> <button onclick="copyText(\'' + botResponse + '\')">复制</button>';

const textElement = processingMessage.querySelector('.text');

for (let i = 0; i < botResponse.length; i++) {

await new Promise(resolve => setTimeout(resolve, 50));

textElement.innerHTML += botResponse[i];

}

} catch (error) {


processingMessage.innerHTML = `<div class="text">发生错误: ${error.message}</div> <button onclick="copyText(\'发生错误: ${error.message}\')">复制</button>`;

} finally {

// 自动滚动到最新消息

chatBox.scrollTop = chatBox.scrollHeight;

}

}



function clearChat() {

const chatBox = document.getElementById('chat-box');

chatBox.innerHTML = '';

}



function copyText(text) {

navigator.clipboard.writeText(text).then(() => {

alert('文本已复制到剪贴板');

}).catch(err => {

console.error('无法复制文本: ', err);

});

}

</script>

</body>

</html>
