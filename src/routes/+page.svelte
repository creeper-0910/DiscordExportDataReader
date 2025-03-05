<script lang="ts">
  import { BaseDirectory, exists, mkdir, readDir, readTextFile } from '@tauri-apps/plugin-fs';
  import { onMount } from "svelte";
  const roomList = [];
  const roomData = {
    chName: "",
    messages: []
  }
  const messageData = {
    userName: "",
    userIcon: "",
    message: ""
  }
  onMount(async () => {
  if(!await exists('ExportData', { baseDir: BaseDirectory.Resource })) {
    await mkdir('ExportData', {baseDir: BaseDirectory.Resource})
  } else {
    console.log("exist");
  }
  const entries = await readDir('ExportData', { baseDir: BaseDirectory.Resource });
  for (const entry of entries) {
    if (!entry.isDirectory) {
      console.log(`Entry: ${entry.name}`);
      const contents = await readTextFile(`ExportData/${entry.name}`, { baseDir: BaseDirectory.Resource });
      const parsedData = JSON.parse(contents);
      roomData.chName = parsedData.channel.name;
      for (const messages of parsedData.messages) {
        messageData.userName = messages.author.name;
        messageData.userIcon = messages.author.avaterUrl;
        messageData.message = messages.content;
        roomData.messages.push(structuredClone(messageData));
        messageData.userName = "";
        messageData.userIcon = "";
        messageData.message = "";
      }
      roomList.push(structuredClone(roomData));
      roomData.chName = "";
      roomData.messages = [];
    }
  }
  })
</script>

<main class="bg-gray-600 h-screen w-screen">
<div></div>
</main>