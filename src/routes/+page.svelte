<script lang="ts">
  import { convertFileSrc } from "@tauri-apps/api/core";
  import { resourceDir } from "@tauri-apps/api/path";
  import { BaseDirectory, exists, mkdir, readDir, readTextFile } from '@tauri-apps/plugin-fs';
  import { onMount } from "svelte";
  import "../app.css";
  const roomList = $state([]);
  const roomData = {
    chName: "",
    messages: []
  }
  const messageData = {
    userName: "",
    userIcon: "",
    date: "",
    message: "",
    attachments: []
  }
  let index = $state(0);
  let resource;
  let searchValue = $state("");
  onMount(async () => {
  if(!await exists('ExportData', { baseDir: BaseDirectory.Resource })) {
    await mkdir('ExportData', {baseDir: BaseDirectory.Resource})
  } else {
    console.log("exist");
  }
  resource = await resourceDir();
  const entries = await readDir('ExportData', { baseDir: BaseDirectory.Resource });
  for (const entry of entries) {
    if (!entry.isDirectory) {
      console.log(`Entry: ${entry.name}`);
      const contents = await readTextFile(`ExportData/${entry.name}`, { baseDir: BaseDirectory.Resource });
      const parsedData = await JSON.parse(contents);
      roomData.chName = parsedData.channel.name;
      for (const messages of parsedData.messages) {
        const date = new Date(messages.timestamp);
        let replacedMessage = messages.content;
        for(const mentions of messages.mentions) {
          replacedMessage = replacedMessage.replace(`<@${mentions.id}>`,`<p class="text-blue-500" >@${mentions.nickname}</p>`)
          replacedMessage = replacedMessage.replace(`<@!${mentions.id}>`,`<p class="text-blue-500" >@${mentions.nickname}</p>`)
        }
        messageData.userName = messages.author.name;
        messageData.userIcon = messages.author.avatarUrl;
        messageData.date = date.toLocaleDateString();
        messageData.message = replacedMessage;
        messageData.attachments = messages.attachments;
        roomData.messages.push(structuredClone(messageData));
        messageData.userName = "";
        messageData.userIcon = "";
        messageData.date = "";
        messageData.message = "";
        messageData.attachments = [];
      }
      roomList.push(structuredClone(roomData));
      roomData.chName = "";
      roomData.messages = [];
    }
  }
  const searchBox = document.getElementById("searchBar");
  searchBox?.addEventListener("change",(e)=>{
    searchValue = e.target.value;
  })
  })
</script>
<div class="navbar bg-base-200 shadow-sm justify-between">
  <a class="btn btn-ghost text-xl">Discord Exporter Viewer</a>
  <input type="text" class="input" id="searchBar" placeholder="検索" />
</div>
<main class="bg-base-50 h-screen w-screen flex overflow-hidden">
  {#if roomList[index]?.messages?.length > 0}
  <div class="min-w-max max-w-max border-r-1 border-base-200 bg-base-200 drawer drawer-open overflow-y-auto overflow-x-hidden">
    <ul class="menu min-w-max max-w-max drawer-content">
      {#each roomList as channel,key}
      <li><button onclick={() => {
        index=key
        }}>{channel.chName}</button></li>
      {/each}
    </ul>
  </div>
  <ul class="list overflow-y-auto overflow-x-hidden w-full">
  {#each roomList[index].messages as message}
  {#if message.message.includes(searchValue)}
  <li class="list-row">
    <div><img class="size-10 rounded-full" src={ convertFileSrc(`${resource}/ExportData/${message.userIcon}`) }/></div>
    <div>
      <div class="text-md flex gap-2 items-center">{message.userName}<p class="text-xs text-gray-400">{message.date}</p></div>
      <p class="list-col-wrap whitespace-pre-wrap">
        {@html message.message}
      </p>
      {#if message.attachments.length > 0}
      {#each message.attachments as attachment}
      <div class="grid grid-cols-2 gap-2 w-1/6"><img class="object-scale-down" src={ convertFileSrc(`${resource}/ExportData/${attachment.url}`) }/></div>
      {/each}
      {/if}
    </div>
  </li>
  {/if}
  {/each}
  </ul>
  {/if}
</main>