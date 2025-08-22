// utils/audioStreamPlayer.ts
type EventName = "start" | "stop" | "end";

export class AudioStreamPlayer {
  private audio: HTMLAudioElement | null = null;
  private mediaSource: MediaSource | null = null;
  private sourceBuffer: SourceBuffer | null = null;
  private queue: Uint8Array[] = [];
  private speaking = false;
  private eventListeners: Record<EventName, (() => void)[]> = {
    start: [],
    stop: [],
    end: [],
  };

  constructor(private apiUrl: string) {}

  /** Event emitter: subscribe */
  on(event: EventName, handler: () => void) {
    this.eventListeners[event].push(handler);
  }

  /** Event emitter: unsubscribe */
  off(event: EventName, handler: () => void) {
    this.eventListeners[event] = this.eventListeners[event].filter(h => h !== handler);
  }

  /** Event emitter: notify listeners */
  private emit(event: EventName) {
    this.eventListeners[event].forEach(handler => handler());
  }

  /** Start streaming TTS */
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
    };

    this.mediaSource.addEventListener("sourceopen", async () => {
      if (!this.mediaSource) return;
      this.sourceBuffer = this.mediaSource.addSourceBuffer('audio/mpeg');

      this.sourceBuffer.addEventListener("updateend", () => {
        if (this.queue.length > 0 && this.sourceBuffer && !this.sourceBuffer.updating) {
          this.sourceBuffer.appendBuffer(this.queue.shift()!);
        }
      });

      await this.streamAudio(text);
    });

    await this.audio.play();
  }

  /** Stream audio chunks from API */
  private async streamAudio(text: string) {
    const resp = await fetch(this.apiUrl, {
      method: "POST",
      body: JSON.stringify({ text }),
      headers: { "Content-Type": "application/json" },
    });

    if (!resp.body) return;

    const reader = resp.body.getReader();
    const decoder = new TextDecoder();

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      const chunk = decoder.decode(value, { stream: true });
      const lines = chunk.split("\n").filter(l => l.trim().startsWith("data:"));

      for (const line of lines) {
        const data = JSON.parse(line.replace("data: ", ""));
        if (data.type === "speech.audio.delta") {
          const audioData = Uint8Array.from(atob(data.audio), c => c.charCodeAt(0));
          if (this.sourceBuffer && !this.sourceBuffer.updating) {
            this.sourceBuffer.appendBuffer(audioData);
          } else {
            this.queue.push(audioData);
          }
        } else if (data.type === "speech.audio.done") {
          this.mediaSource?.endOfStream();
        }
      }
    }
  }

  /** Stop playback immediately */
  stop() {
    if (this.audio) {
      this.audio.pause();
      this.audio.src = "";
      this.audio = null;
    }
    this.mediaSource = null;
    this.sourceBuffer = null;
    this.queue = [];
    this.speaking = false;
    this.emit("stop");
  }

  /** Check if audio is currently playing */
  isSpeaking() {
    return this.speaking;
  }
}
