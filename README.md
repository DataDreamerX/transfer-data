type Listener = (...args: any[]) => void;

export class AudioStreamer {
  private audio: HTMLAudioElement;
  private mediaSource: MediaSource | null = null;
  private sourceBuffer: SourceBuffer | null = null;
  private speaking = false;
  private listeners: Record<string, Listener[]> = {};

  constructor() {
    this.audio = new Audio();
    this.audio.autoplay = true;
  }

  private emit(event: string, ...args: any[]) {
    (this.listeners[event] || []).forEach((fn) => fn(...args));
  }

  on(event: string, fn: Listener) {
    if (!this.listeners[event]) this.listeners[event] = [];
    this.listeners[event].push(fn);
  }

  off(event: string, fn: Listener) {
    if (!this.listeners[event]) return;
    this.listeners[event] = this.listeners[event].filter((f) => f !== fn);
  }

  async streamAudio(url: string, text: string) {
    this.stop(); // reset if already playing

    this.mediaSource = new MediaSource();
    this.audio.src = URL.createObjectURL(this.mediaSource);

    this.mediaSource.addEventListener("sourceopen", async () => {
      this.sourceBuffer = this.mediaSource!.addSourceBuffer('audio/mpeg');

      // ✅ attach `onended` once per playback
      this.audio.onended = () => {
        if (this.speaking) {
          this.speaking = false;
          this.emit("end");
        }
      };

      const response = await fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ text }),
      });

      const reader = response.body?.getReader();
      if (!reader) return;

      this.speaking = true;
      this.emit("start");

      const pump = async () => {
        const { value, done } = await reader.read();
        if (done) {
          this.mediaSource?.endOfStream();
          return;
        }
        if (value) {
          const chunk = new TextDecoder().decode(value).trim();
          if (chunk.startsWith("data: ")) {
            const data = JSON.parse(chunk.slice(6));
            if (data.type === "speech.audio.delta" && data.audio) {
              const audioChunk = Uint8Array.from(atob(data.audio), c => c.charCodeAt(0));
              await this.appendBuffer(audioChunk);
            }
          }
        }
        pump();
      };

      pump();
    });
  }

  private appendBuffer(chunk: Uint8Array) {
    return new Promise<void>((resolve, reject) => {
      if (!this.sourceBuffer) return resolve();
      const sb = this.sourceBuffer;
      const append = () => {
        try {
          sb.appendBuffer(chunk);
          sb.removeEventListener("updateend", append);
          resolve();
        } catch (e) {
          reject(e);
        }
      };
      if (!sb.updating) {
        sb.addEventListener("updateend", append, { once: true });
        sb.appendBuffer(chunk);
      } else {
        sb.addEventListener("updateend", append, { once: true });
      }
    });
  }

  stop() {
    if (this.audio) {
      this.audio.pause();
      this.audio.src = "";
    }
    this.speaking = false;
    this.mediaSource = null;
    this.sourceBuffer = null;
    this.emit("stop");
  }

  isSpeaking() {
    return this.speaking;
  }
}
