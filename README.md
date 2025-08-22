"use client";

import { useRef, useState } from "react";

export default function Home() {
  const [text, setText] = useState("Hello from streaming TTS!");
  const audioRef = useRef<HTMLAudioElement | null>(null);

  const handleSpeak = async () => {
    const mediaSource = new MediaSource();
    const audioEl = new Audio();
    audioEl.src = URL.createObjectURL(mediaSource);
    audioEl.play();
    audioRef.current = audioEl;

    mediaSource.addEventListener("sourceopen", async () => {
      const sourceBuffer = mediaSource.addSourceBuffer('audio/mpeg'); // mp3 chunks

      const res = await fetch("/api/tts", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ text }),
      });

      const reader = res.body!.getReader();
      const decoder = new TextDecoder("utf-8");
      let buffer = "";

      while (true) {
        const { done, value } = await reader.read();
        if (done) {
          mediaSource.endOfStream();
          break;
        }

        buffer += decoder.decode(value, { stream: true });
        const parts = buffer.split("\n\n");
        buffer = parts.pop() || "";

        for (const part of parts) {
          if (!part.startsWith("data:")) continue;
          const jsonStr = part.replace(/^data:\s*/, "");
          if (!jsonStr || jsonStr === "[DONE]") continue;

          const event = JSON.parse(jsonStr);

          if (event.type === "speech.audio.delta") {
            const chunk = base64ToUint8Array(event.audio);
            // append to buffer when it's ready
            if (!sourceBuffer.updating) {
              sourceBuffer.appendBuffer(chunk);
            } else {
              sourceBuffer.addEventListener("updateend", function handler() {
                sourceBuffer.removeEventListener("updateend", handler);
                sourceBuffer.appendBuffer(chunk);
              });
            }
          }
        }
      }
    });
  };

  function base64ToUint8Array(base64: string) {
    const binary = atob(base64);
    const len = binary.length;
    const bytes = new Uint8Array(len);
    for (let i = 0; i < len; i++) {
      bytes[i] = binary.charCodeAt(i);
    }
    return bytes;
  }

  return (
    <div className="p-8">
      <h1 className="text-2xl mb-4">🔊 Streaming TTS</h1>
      <textarea
        className="border p-2 w-full"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={handleSpeak}
        className="mt-4 px-4 py-2 bg-blue-600 text-white rounded"
      >
        Speak
      </button>
    </div>
  );
}
