// utils/audioStreamPlayer.ts
export class AudioStreamPlayer {
  private mediaSource: MediaSource | null = null;
  private audio: HTMLAudioElement | null = null;
  private sourceBuffer: SourceBuffer | null = null;
  private queue: Uint8Array[] = [];
  private isAppending = false;
  private speaking = false;

  constructor(private apiUrl: string) {}

  async start(text: string) {
    this.stop(); // ensure previous session is cleared

    this.mediaSource = new MediaSource();
    this.audio = new Audio();
    this.audio.src = URL.createObjectURL(this.mediaSource);

    this.audio.onplay = () => {
      this.speaking = true;
    };

    this.audio.onpause = () => {
      this.speaking = false;
    };

    this.audio.onended = () => {
      this.speaking = false;
    };

    this.mediaSource.addEventListener("sourceopen", () => {
      if (!this.mediaSource) return;

      this.sourceBuffer = this.mediaSource.addSourceBuffer("audio/mpeg");

      this.sourceBuffer.addEventListener("updateend", () => {
        this.isAppending = false;
        this.feedBuffer();
      });

      this.streamFromApi(text);
    });

    await this.audio.play();
  }

  private async streamFromApi(text: string) {
    const response = await fetch(this.apiUrl, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ text }),
    });

    const reader = response.body?.getReader();
    if (!reader) return;

    const decoder = new TextDecoder("utf-8");
    let buffer = "";

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      buffer += decoder.decode(value, { stream: true });
      const parts = buffer.split("\n\n");
      buffer = parts.pop() || "";

      for (const part of parts) {
        if (!part.startsWith("data:")) continue;
        const jsonStr = part.replace(/^data:\s*/, "");
        if (!jsonStr.trim()) continue;

        const event = JSON.parse(jsonStr);

        if (event.type === "speech.audio.delta") {
          const chunk = this.base64ToUint8Array(event.audio);
          this.enqueueChunk(chunk);
        } else if (event.type === "speech.audio.done") {
          try {
            this.mediaSource?.endOfStream();
          } catch {}
        }
      }
    }
  }

  private enqueueChunk(chunk: Uint8Array) {
    this.queue.push(chunk);
    this.feedBuffer();
  }

  private feedBuffer() {
    if (!this.sourceBuffer || this.isAppending) return;
    if (this.queue.length === 0) return;

    this.isAppending = true;
    const chunk = this.queue.shift();
    if (chunk) {
      try {
        this.sourceBuffer.appendBuffer(chunk);
      } catch (err) {
        console.error("appendBuffer error:", err);
        this.isAppending = false;
      }
    }
  }

  stop() {
    if (this.audio) {
      this.audio.pause();
      this.audio.src = "";
      this.audio = null;
    }
    if (this.mediaSource) {
      try {
        this.mediaSource.endOfStream();
      } catch {}
      this.mediaSource = null;
    }
    this.sourceBuffer = null;
    this.queue = [];
    this.isAppending = false;
    this.speaking = false;
  }

  isSpeaking(): boolean {
    return this.speaking;
  }

  private base64ToUint8Array(base64: string): Uint8Array {
    const binary = atob(base64);
    const len = binary.length;
    const bytes = new Uint8Array(len);
    for (let i = 0; i < len; i++) {
      bytes[i] = binary.charCodeAt(i);
    }
    return bytes;
  }
}
