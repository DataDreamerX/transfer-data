"use client";

import { useRef, useState } from "react";

export default function Home() {
  const [text, setText] = useState("Hello, streaming world!");
  const [audioContext, setAudioContext] = useState<AudioContext | null>(null);

  // keep track of scheduling time (not in React state!)
  const nextStartTimeRef = useRef(0);

  const handleSpeak = async () => {
    const ctx = audioContext || new AudioContext();
    if (!audioContext) setAudioContext(ctx);

    // reset scheduling before new playback
    nextStartTimeRef.current = ctx.currentTime;

    const res = await fetch("/api/tts", {
      method: "POST",
      body: JSON.stringify({ text }),
      headers: { "Content-Type": "application/json" },
    });

    const reader = res.body!.getReader();
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
        if (!jsonStr || jsonStr === "[DONE]") continue;

        const event = JSON.parse(jsonStr);

        if (event.type === "speech.audio.delta") {
          const audioData = base64ToArrayBuffer(event.audio);
          schedulePlayback(ctx, audioData);
        } else if (event.type === "speech.audio.done") {
          console.log("✅ Stream finished");
        }
      }
    }
  };

  function base64ToArrayBuffer(base64: string) {
    const binary = atob(base64);
    const len = binary.length;
    const bytes = new Uint8Array(len);
    for (let i = 0; i < len; i++) {
      bytes[i] = binary.charCodeAt(i);
    }
    return bytes.buffer;
  }

  async function schedulePlayback(ctx: AudioContext, chunk: ArrayBuffer) {
    try {
      const audioBuffer = await ctx.decodeAudioData(chunk);
      const source = ctx.createBufferSource();
      source.buffer = audioBuffer;
      source.connect(ctx.destination);

      // queue after the previous chunk
      const startAt = Math.max(ctx.currentTime, nextStartTimeRef.current);
      source.start(startAt);

      // update the "next free time"
      nextStartTimeRef.current = startAt + audioBuffer.duration;
    } catch (err) {
      console.error("Decode/play error:", err);
    }
  }

  return (
    <div className="p-8">
      <h1 className="text-2xl mb-4">🔊 TTS Streaming Test</h1>
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
