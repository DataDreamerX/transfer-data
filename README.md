// app/api/text-to-speech/route.js
import { NextResponse } from 'next/server';

// Function to convert L16 PCM to WAV (same as above)
function convertL16ToWav(inputBuffer, sampleRate, numChannels, bitsPerSample) {
  const HEADER_LENGTH = 44;
  const dataSize = inputBuffer.length;
  const fileSize = dataSize + HEADER_LENGTH;

  const buffer = new ArrayBuffer(fileSize);
  const view = new DataView(buffer);

  // RIFF header
  writeString(view, 0, 'RIFF');
  view.setUint32(4, fileSize - 8, true);
  writeString(view, 8, 'WAVE');

  // FMT sub-chunk
  writeString(view, 12, 'fmt ');
  view.setUint32(16, 16, true); // Subchunk1Size for PCM
  view.setUint16(20, 1, true); // AudioFormat (1 for PCM)
  view.setUint16(22, numChannels, true);
  view.setUint32(24, sampleRate, true);
  view.setUint32(28, sampleRate * numChannels * bitsPerSample / 8, true); // ByteRate
  view.setUint16(32, numChannels * bitsPerSample / 8, true); // BlockAlign
  view.setUint16(34, bitsPerSample, true);

  // Data sub-chunk
  writeString(view, 36, 'data');
  view.setUint32(40, dataSize, true);

  // Write PCM data
  new Uint8Array(buffer, HEADER_LENGTH).set(new Uint8Array(inputBuffer));

  return Buffer.from(buffer);
}

function writeString(view, offset, string) {
  for (let i = 0; i < string.length; i++) {
    view.setUint8(offset + i, string.charCodeAt(i));
  }
}


export async function POST(req) {
  const { text } = await req.json();

  if (!text) {
    return NextResponse.json({ message: 'Text is required.' }, { status: 400 });
  }

  const GEMINI_API_KEY = process.env.GEMINI_API_KEY;
  const GEMINI_MODEL_ID = "gemini-2.5-flash-preview-tts"; // Or another suitable TTS model

  try {
    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/${GEMINI_MODEL_ID}:generateContent?key=${GEMINI_API_KEY}`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          contents: [
            {
              role: 'user',
              parts: [
                {
                  text: text,
                },
              ],
            },
          ],
          generationConfig: {
            responseModalities: ['AUDIO'],
            speechConfig: {
              multiSpeakerVoiceConfig: {
                speakerVoiceConfigs: [
                  {
                    speaker: 'SPEAKER_0',
                    languageCode: 'en-US',
                  },
                ],
              },
            },
          },
        }),
      }
    );

    if (!response.ok) {
      const errorData = await response.json();
      console.error('Gemini API error:', errorData);
      return NextResponse.json({ message: 'Failed to generate speech from Gemini API', error: errorData }, { status: response.status });
    }

    const data = await response.json();
    const audioData = data.candidates[0]?.content?.parts[0]?.inlineData?.data;
    const mimeType = data.candidates[0]?.content?.parts[0]?.inlineData?.mimeType;

    if (!audioData) {
      return NextResponse.json({ message: 'No audio data received from Gemini API.' }, { status: 500 });
    }

    const audioBuffer = Buffer.from(audioData, 'base64');

    let playableAudioBuffer = audioBuffer;
    let playableMimeType = mimeType;

    if (mimeType.includes('audio/L16') && mimeType.includes('codec=pcm') && mimeType.includes('rate=24000')) {
        playableAudioBuffer = convertL16ToWav(audioBuffer, 24000, 1, 16);
        playableMimeType = 'audio/wav';
    } else if (mimeType.includes('audio/mpeg')) {
        playableMimeType = 'audio/mp3';
    }

    return new Response(playableAudioBuffer, {
      headers: {
        'Content-Type': playableMimeType,
      },
      status: 200,
    });

  } catch (error) {
    console.error('Error in text-to-speech API route:', error);
    return NextResponse.json({ message: 'Internal Server Error', error: error.message }, { status: 500 });
  }
}
