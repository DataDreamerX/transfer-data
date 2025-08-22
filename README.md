async start(text: string) {
  this.stop(); // ensure clean state

  this.mediaSource = new MediaSource();
  this.audio = new Audio();
  this.audio.src = URL.createObjectURL(this.mediaSource);

  this.audio.onplay = () => {
    this.speaking = true;
    this.emit("start");
  };
  this.audio.onpause = () => {
    this.speaking = false;
    this.emit("stop");
  };
  this.audio.onended = () => {
    this.speaking = false;
    this.emit("end");
    this.clear(); // cleanup after finished playing
  };

  // Wait until source is open
  this.mediaSource.addEventListener("sourceopen", async () => {
    if (!this.mediaSource) return;
    this.sourceBuffer = this.mediaSource.addSourceBuffer("audio/mpeg");

    this.sourceBuffer.addEventListener("updateend", () => {
      if (this.queue.length > 0 && this.sourceBuffer && !this.sourceBuffer.updating) {
        this.sourceBuffer.appendBuffer(this.queue.shift()!);
      }
    });

    // 🚀 Start streaming
    await this.streamAudio(text);

    // ✅ Only play after at least one chunk is buffered
    const tryPlay = () => {
      if (this.audio && this.audio.readyState >= 2) {
        this.audio.play().catch(err => console.error("Play failed:", err));
      } else {
        setTimeout(tryPlay, 50); // retry until data is ready
      }
    };
    tryPlay();
  });
}
