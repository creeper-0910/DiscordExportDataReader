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
  const attachmentsList = ["png","jpg","gif","apng","webp","tiff"];
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
        messageData.userName = messages.author.nickname;
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
<main class="bg-base-50 h-[calc(100vh-4rem)] w-screen flex overflow-hidden grow">
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
  <div class=" overflow-y-auto overflow-x-hidden">
  <ul class="list w-full min-h-max">
  {#each roomList[index].messages as message}
  {#if message.message.includes(searchValue) || message.userName.includes(searchValue) || message.date.includes(searchValue)}
  <li class="list-row h-max">
    <div><img class="size-10 rounded-full" src={ convertFileSrc(`${resource}/ExportData/${message.userIcon.replace("\\","/")}`) }/></div>
    <div>
      <div class="text-md flex gap-2 items-center">{message.userName}<p class="text-xs text-gray-400">{message.date}</p></div>
      <p class="list-col-wrap whitespace-pre-wrap">
        {@html message.message}
      </p>
      {#if message.attachments.length > 0}
      {#each message.attachments as attachment}
      {#if attachmentsList.includes(attachment.fileName.split('.').pop())}
      <div class="grid grid-cols-2 gap-2 w-1/6 min-w-md"><img class="object-scale-down" src={ convertFileSrc(`${resource}/ExportData/${attachment.url.replace("\\","/")}`) }/></div>
      {:else}
      <div class="card bg-base-200 w-max shadow-sm">
        <div class="card-body flex flex-row min-w-sm items-center">
          <div class="flex-auto text-md">{attachment.fileName}</div>
          <div class="flex-none">
            <button class="hover:bg-base-300 p-2 rounded-lg" onclick={location.href= convertFileSrc(`${resource}/ExportData/${attachment.url.replace("\\","/")}`)}>
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
              <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
            </svg>
            </button>
          </div>
        </div>
      </div>
      {/if}
      {/each}
      {/if}
    </div>
  </li>
  {/if}
  {/each}
  </ul>
  </div>
  {/if}
</main>