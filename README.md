"use client"

import type React from "react"

import { useState, useRef, useEffect } from "react"
import { Mic, MicOff, Volume2, VolumeX, Send, Bot } from "lucide-react"

// Gemini API configuration (replace with your actual API key)
const GEMINI_API_KEY = "your-gemini-api-key-here"
const GEMINI_API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent"
const GEMINI_AUDIO_API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro-vision:generateContent"

interface Message {
  id: string
  text: string
  sender: "user" | "bot"
  timestamp: Date
  isVoice?: boolean
}

export default function VoiceChatBot() {
  const [messages, setMessages] = useState<Message[]>([
    {
      id: "1",
      text: "Hello! I'm your voice assistant. You can speak to me or type your message.",
      sender: "bot",
      timestamp: new Date(),
    },
  ])
  const [inputText, setInputText] = useState("")
  const [isListening, setIsListening] = useState(false)
  const [isSpeaking, setIsSpeaking] = useState(false)
  const [isSupported, setIsSupported] = useState(false)

  const recognitionRef = useRef<any>(null)
  const synthesisRef = useRef<SpeechSynthesis | null>(null)
  const messagesEndRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    // Check for browser support
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
    const speechSynthesis = window.speechSynthesis

    if (SpeechRecognition && speechSynthesis) {
      setIsSupported(true)
      recognitionRef.current = new SpeechRecognition()
      synthesisRef.current = speechSynthesis

      // Configure speech recognition
      if (recognitionRef.current) {
        recognitionRef.current.continuous = true // Changed from false to true
        recognitionRef.current.interimResults = true // Changed from false to true
        recognitionRef.current.lang = "en-US"

        recognitionRef.current.onresult = async (event) => {
          let finalTranscript = ""
          let interimTranscript = ""

          for (let i = event.resultIndex; i < event.results.length; i++) {
            const transcript = event.results[i][0].transcript
            if (event.results[i].isFinal) {
              finalTranscript += transcript
            } else {
              interimTranscript += transcript
            }
          }

          if (finalTranscript) {
            // Option to enhance transcription with Gemini (when audio API is available)
            try {
              // For now, use the browser's transcription directly
              handleSendMessage(finalTranscript, true)
            } catch (error) {
              console.error("Enhanced transcription failed:", error)
              handleSendMessage(finalTranscript, true)
            }
          }

          // Update input with interim results for visual feedback
          if (interimTranscript && isListening) {
            setInputText(interimTranscript)
          }
        }

        recognitionRef.current.onerror = (event) => {
          console.error("Speech recognition error:", event.error)
          if (event.error === "no-speech" || event.error === "audio-capture") {
            // Don't stop listening for these errors, just continue
            return
          }
          setIsListening(false)
        }

        recognitionRef.current.onend = () => {
          if (isListening) {
            // Restart recognition if we're still supposed to be listening
            setTimeout(() => {
              if (recognitionRef.current && isListening) {
                recognitionRef.current.start()
              }
            }, 100)
          }
        }
      }
    }
  }, [])

  useEffect(() => {
    scrollToBottom()
  }, [messages])

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" })
  }

  const startListening = () => {
    if (recognitionRef.current && !isListening) {
      setIsListening(true)
      recognitionRef.current.start()
    }
  }

  const stopListening = () => {
    if (recognitionRef.current && isListening) {
      recognitionRef.current.stop()
      setIsListening(false)
    }
  }

  const speakText = (text: string) => {
    if (synthesisRef.current && !isSpeaking) {
      const utterance = new SpeechSynthesisUtterance(text)
      utterance.rate = 0.9
      utterance.pitch = 1
      utterance.volume = 0.8

      utterance.onstart = () => setIsSpeaking(true)
      utterance.onend = () => setIsSpeaking(false)
      utterance.onerror = () => setIsSpeaking(false)

      synthesisRef.current.speak(utterance)
    }
  }

  const stopSpeaking = () => {
    if (synthesisRef.current) {
      synthesisRef.current.cancel()
      setIsSpeaking(false)
    }
  }

  const handleSendMessage = async (text: string, isVoice = false) => {
    if (!text.trim()) return

    const userMessage: Message = {
      id: Date.now().toString(),
      text: text.trim(),
      sender: "user",
      timestamp: new Date(),
      isVoice,
    }

    setMessages((prev) => [...prev, userMessage])
    setInputText("") // Clear input including interim results

    // Call Gemini API for bot response
    try {
      const botResponse = await callGeminiAPI(text.trim())
      const botMessage: Message = {
        id: (Date.now() + 1).toString(),
        text: botResponse,
        sender: "bot",
        timestamp: new Date(),
      }

      setMessages((prev) => [...prev, botMessage])

      // Convert bot response to speech using Gemini TTS
      if (isVoice) {
        setTimeout(() => convertTextToSpeechGemini(botResponse), 500)
      }
    } catch (error) {
      console.error("Error calling Gemini API:", error)
      // Fallback to original bot response
      const fallbackResponse = generateBotResponse(text.trim())
      const botMessage: Message = {
        id: (Date.now() + 1).toString(),
        text: fallbackResponse,
        sender: "bot",
        timestamp: new Date(),
      }

      setMessages((prev) => [...prev, botMessage])

      if (isVoice) {
        setTimeout(() => speakText(fallbackResponse), 500)
      }
    }
  }

  // Call Gemini API for text generation
  const callGeminiAPI = async (userText: string): Promise<string> => {
    try {
      const response = await fetch(`${GEMINI_API_URL}?key=${GEMINI_API_KEY}`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          contents: [
            {
              parts: [
                {
                  text: `You are a helpful Humana health assistant. Please respond to this user message in a friendly and informative way: "${userText}"`,
                },
              ],
            },
          ],
          generationConfig: {
            temperature: 0.7,
            topK: 40,
            topP: 0.95,
            maxOutputTokens: 1024,
          },
        }),
      })

      if (!response.ok) {
        throw new Error(`Gemini API error: ${response.status}`)
      }

      const data = await response.json()
      return data.candidates?.[0]?.content?.parts?.[0]?.text || "I'm sorry, I couldn't process your request right now."
    } catch (error) {
      console.error("Gemini API call failed:", error)
      throw error
    }
  }

  // Convert audio to text using Gemini (placeholder for future audio API)
  const convertAudioToTextGemini = async (audioBlob: Blob): Promise<string> => {
    try {
      // This is a placeholder - Gemini's audio-to-text API is still in development
      // For now, we'll continue using the browser's speech recognition
      // When Gemini audio API is available, implement it here

      const formData = new FormData()
      formData.append("audio", audioBlob)

      const response = await fetch(`${GEMINI_AUDIO_API_URL}?key=${GEMINI_API_KEY}`, {
        method: "POST",
        body: formData,
      })

      if (!response.ok) {
        throw new Error(`Gemini Audio API error: ${response.status}`)
      }

      const data = await response.json()
      return data.transcript || ""
    } catch (error) {
      console.error("Gemini audio-to-text failed:", error)
      throw error
    }
  }

  // Convert text to speech using Gemini (placeholder for future TTS API)
  const convertTextToSpeechGemini = async (text: string) => {
    try {
      // This is a placeholder - Gemini's TTS API is still in development
      // For now, we'll continue using the browser's speech synthesis
      // When Gemini TTS API is available, implement it here

      const response = await fetch(`${GEMINI_API_URL}/tts?key=${GEMINI_API_KEY}`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          text: text,
          voice: {
            languageCode: "en-US",
            name: "en-US-Standard-A",
          },
          audioConfig: {
            audioEncoding: "MP3",
          },
        }),
      })

      if (!response.ok) {
        throw new Error(`Gemini TTS API error: ${response.status}`)
      }

      const data = await response.json()
      const audioContent = data.audioContent

      // Play the audio
      const audio = new Audio(`data:audio/mp3;base64,${audioContent}`)
      audio.play()
    } catch (error) {
      console.error("Gemini TTS failed, falling back to browser TTS:", error)
      // Fallback to browser speech synthesis
      speakText(text)
    }
  }

  const generateBotResponse = (userText: string): string => {
    const responses = [
      "That's an interesting point. Can you tell me more about that?",
      "I understand what you're saying. How can I help you with that?",
      "Thanks for sharing that with me. What would you like to know?",
      "I see. Is there anything specific you'd like assistance with?",
      "That makes sense. What other questions do you have?",
    ]

    if (userText.toLowerCase().includes("hello") || userText.toLowerCase().includes("hi")) {
      return "Hello! It's great to hear from you. How can I assist you today?"
    }

    if (userText.toLowerCase().includes("help")) {
      return "I'm here to help! You can ask me questions, and I'll do my best to provide useful information."
    }

    return responses[Math.floor(Math.random() * responses.length)]
  }

  const handleKeyPress = (e: React.KeyboardEvent) => {
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault()
      handleSendMessage(inputText)
    }
  }

  return (
    <div
      className="nds-container-fluid"
      style={{ height: "100vh", background: "linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%)" }}
    >
      <div
        className="nds-row nds-justify-content-center nds-align-items-center"
        style={{ minHeight: "100vh", padding: "20px 0" }}
      >
        <div className="nds-col-12 nds-col-md-10 nds-col-lg-8 nds-col-xl-6">
          {/* Main Chat Container */}
          <div
            className="chat-container nds-card"
            style={{
              height: "85vh",
              maxHeight: "800px",
              borderRadius: "24px",
              border: "none",
              boxShadow: "0 20px 60px rgba(0,0,0,0.1), 0 8px 25px rgba(0,0,0,0.08)",
              overflow: "hidden",
              background: "white",
            }}
          >
            {/* Enhanced Header */}
            <div
              className="nds-card-header"
              style={{
                background: "linear-gradient(135deg, #00a94f 0%, #00c851 100%)",
                border: "none",
                padding: "20px 24px",
                color: "white",
              }}
            >
              <div className="nds-d-flex nds-align-items-center nds-justify-content-between">
                <div className="nds-d-flex nds-align-items-center">
                  <div
                    className="bot-avatar"
                    style={{
                      width: "48px",
                      height: "48px",
                      borderRadius: "50%",
                      background: "rgba(255,255,255,0.2)",
                      display: "flex",
                      alignItems: "center",
                      justifyContent: "center",
                      marginRight: "16px",
                      backdropFilter: "blur(10px)",
                    }}
                  >
                    <Bot size={24} color="white" />
                  </div>
                  <div>
                    <h4 className="nds-mb-0" style={{ fontWeight: "600", fontSize: "18px" }}>
                      Humana Assistant
                    </h4>
                    <div className="nds-d-flex nds-align-items-center nds-mt-1">
                      <div
                        style={{
                          width: "8px",
                          height: "8px",
                          borderRadius: "50%",
                          backgroundColor: isListening ? "#ff4757" : isSpeaking ? "#ffa502" : "#2ed573",
                          marginRight: "8px",
                          animation: isListening || isSpeaking ? "pulse 1.5s infinite" : "none",
                        }}
                      ></div>
                      <small style={{ opacity: "0.9", fontSize: "13px" }}>
                        {isListening ? "Listening..." : isSpeaking ? "Speaking..." : "Ready to help"}
                      </small>
                    </div>
                  </div>
                </div>

                <div className="nds-d-flex nds-align-items-center">
                  {!isSupported && (
                    <span
                      className="nds-badge"
                      style={{
                        backgroundColor: "rgba(255,193,7,0.2)",
                        color: "#fff",
                        border: "1px solid rgba(255,255,255,0.3)",
                        fontSize: "11px",
                      }}
                    >
                      Voice Unavailable
                    </span>
                  )}
                </div>
              </div>
            </div>

            {/* Enhanced Messages Area */}
            <div
              className="messages-container"
              style={{
                height: "calc(100% - 180px)",
                overflowY: "auto",
                padding: "24px",
                background: "#fafbfc",
              }}
            >
              {messages.map((message, index) => (
                <div
                  key={message.id}
                  className={`message-wrapper nds-d-flex nds-mb-5`}
                  style={{
                    ...(message.sender === "user"
                      ? {
                          display: "flex",
                          justifyContent: "flex-end",
                          width: "100%",
                          animation: "slideIn 0.3s ease-out 0.1s forwards",
                        }
                      : {
                          animation: `slideIn 0.3s ease-out ${index * 0.1}s both`,
                          justifyContent: "flex-start",
                          width: "100%",
                        }),
                  }}
                >
                  <div
                    className="message-content"
                    style={{
                      maxWidth: "70%",
                      minWidth: "auto",
                      width: "fit-content",
                      position: "relative",
                      display: "flex",
                      flexDirection: "column",
                      alignItems: message.sender === "user" ? "flex-end" : "flex-start",
                    }}
                  >
                    <div
                      className={`message-bubble ${message.sender === "user" ? "user-message" : "bot-message"}`}
                      style={{
                        padding: "16px 20px",
                        borderRadius: message.sender === "user" ? "20px 20px 4px 20px" : "20px 20px 20px 4px",
                        background: message.sender === "user" ? "linear-gradient(135deg, #007bff, #0056b3)" : "white",
                        color: message.sender === "user" ? "white" : "#333",
                        boxShadow:
                          message.sender === "user" ? "0 4px 12px rgba(0,123,255,0.3)" : "0 2px 8px rgba(0,0,0,0.1)",
                        borderWidth: message.sender === "bot" ? "1px" : "0",
                        borderStyle: message.sender === "bot" ? "solid" : "none",
                        borderColor: message.sender === "bot" ? "#e9ecef" : "transparent",
                        position: "relative",
                      }}
                    >
                      {message.isVoice && (
                        <div
                          className="voice-indicator"
                          style={{
                            display: "flex",
                            alignItems: "center",
                            marginBottom: "8px",
                            opacity: "0.8",
                          }}
                        >
                          <Volume2 size={14} style={{ marginRight: "6px" }} />
                          <span style={{ fontSize: "12px", fontWeight: "500" }}>Voice Message</span>
                        </div>
                      )}

                      <p
                        className="nds-mb-0"
                        style={{
                          lineHeight: "1.5",
                          fontSize: "15px",
                          margin: "0",
                        }}
                      >
                        {message.text}
                      </p>

                      <div
                        className="message-time"
                        style={{
                          fontSize: "11px",
                          opacity: "0.7",
                          marginTop: "8px",
                          textAlign: message.sender === "user" ? "right" : "left",
                        }}
                      >
                        {message.timestamp.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })}
                      </div>
                    </div>
                  </div>
                </div>
              ))}
              <div ref={messagesEndRef} />
            </div>

            {/* Enhanced Input Area */}
            <div
              className="input-container"
              style={{
                padding: "20px 24px",
                background: "white",
                borderTop: "1px solid #e9ecef",
              }}
            >
              <div
                className="input-wrapper"
                style={{
                  position: "relative",
                  background: "#f8f9fa",
                  borderRadius: "24px",
                  border: "2px solid #e9ecef",
                  transition: "all 0.2s ease",
                  ...(inputText.trim() && { borderColor: "#00a94f" }),
                }}
              >
                <textarea
                  className="message-input"
                  placeholder={
                    isListening && inputText
                      ? "Listening... (speak now)"
                      : "Type your message or click the mic to speak..."
                  }
                  value={inputText}
                  onChange={(e) => setInputText(e.target.value)}
                  onKeyPress={handleKeyPress}
                  rows={1}
                  style={{
                    width: "100%",
                    border: "none",
                    background: "transparent",
                    padding: "16px 120px 16px 20px",
                    resize: "none",
                    outline: "none",
                    fontSize: "15px",
                    lineHeight: "1.4",
                    fontFamily: "inherit",
                  }}
                />

                <div
                  className="input-controls"
                  style={{
                    position: "absolute",
                    right: "8px",
                    top: "50%",
                    transform: "translateY(-50%)",
                    display: "flex",
                    alignItems: "center",
                    gap: "8px",
                  }}
                >
                  {/* Voice Controls */}
                  {isSupported && (
                    <>
                      <button
                        className={`voice-btn ${isListening ? "listening" : ""}`}
                        onClick={isListening ? stopListening : startListening}
                        disabled={isSpeaking}
                        style={{
                          width: "40px",
                          height: "40px",
                          borderRadius: "50%",
                          border: "none",
                          background: isListening
                            ? "linear-gradient(135deg, #ff4757, #ff3742)"
                            : "linear-gradient(135deg, #00a94f, #00c851)",
                          color: "white",
                          display: "flex",
                          alignItems: "center",
                          justifyContent: "center",
                          cursor: "pointer",
                          transition: "all 0.2s ease",
                          boxShadow: isListening
                            ? "0 0 20px rgba(255, 71, 87, 0.6), 0 0 40px rgba(255, 71, 87, 0.4)"
                            : "0 2px 8px rgba(0,0,0,0.15)",
                          position: "relative",
                          ...(isListening && {
                            animation: "micPulse 1.5s infinite, micGlow 2s infinite alternate",
                            transform: "scale(1.1)",
                          }),
                        }}
                      >
                        <div
                          style={{
                            position: "relative",
                            display: "flex",
                            alignItems: "center",
                            justifyContent: "center",
                          }}
                        >
                          {isListening ? <MicOff size={18} /> : <Mic size={18} />}
                          {isListening && (
                            <>
                              <div
                                className="sound-wave wave-1"
                                style={{
                                  position: "absolute",
                                  width: "60px",
                                  height: "60px",
                                  border: "2px solid rgba(255, 255, 255, 0.3)",
                                  borderRadius: "50%",
                                  animation: "soundWave 1.5s infinite",
                                  animationDelay: "0s",
                                }}
                              />
                              <div
                                className="sound-wave wave-2"
                                style={{
                                  position: "absolute",
                                  width: "80px",
                                  height: "80px",
                                  border: "2px solid rgba(255, 255, 255, 0.2)",
                                  borderRadius: "50%",
                                  animation: "soundWave 1.5s infinite",
                                  animationDelay: "0.5s",
                                }}
                              />
                              <div
                                className="sound-wave wave-3"
                                style={{
                                  position: "absolute",
                                  width: "100px",
                                  height: "100px",
                                  border: "2px solid rgba(255, 255, 255, 0.1)",
                                  borderRadius: "50%",
                                  animation: "soundWave 1.5s infinite",
                                  animationDelay: "1s",
                                }}
                              />
                            </>
                          )}
                        </div>
                      </button>

                      <button
                        className="speaker-btn"
                        onClick={isSpeaking ? stopSpeaking : () => {}}
                        disabled={!isSpeaking}
                        style={{
                          width: "40px",
                          height: "40px",
                          borderRadius: "50%",
                          border: "none",
                          background: isSpeaking ? "linear-gradient(135deg, #ffa502, #ff6348)" : "#e9ecef",
                          color: isSpeaking ? "white" : "#6c757d",
                          display: "flex",
                          alignItems: "center",
                          justifyContent: "center",
                          cursor: isSpeaking ? "pointer" : "not-allowed",
                          transition: "all 0.2s ease",
                          opacity: isSpeaking ? 1 : 0.6,
                          ...(isSpeaking && {
                            animation: "bounce 1s infinite",
                          }),
                        }}
                      >
                        {isSpeaking ? <VolumeX size={18} /> : <Volume2 size={18} />}
                      </button>
                    </>
                  )}

                  {/* Send Button */}
                  <button
                    className="send-btn"
                    onClick={() => handleSendMessage(inputText)}
                    disabled={!inputText.trim()}
                    style={{
                      width: "40px",
                      height: "40px",
                      borderRadius: "50%",
                      border: "none",
                      background: inputText.trim() ? "linear-gradient(135deg, #007bff, #0056b3)" : "#e9ecef",
                      color: inputText.trim() ? "white" : "#6c757d",
                      display: "flex",
                      alignItems: "center",
                      justifyContent: "center",
                      cursor: inputText.trim() ? "pointer" : "not-allowed",
                      transition: "all 0.2s ease",
                      transform: inputText.trim() ? "scale(1)" : "scale(0.95)",
                      boxShadow: inputText.trim() ? "0 2px 8px rgba(0,123,255,0.3)" : "none",
                    }}
                  >
                    <Send size={18} />
                  </button>
                </div>
              </div>

              {!isSupported && (
                <div
                  className="help-text"
                  style={{
                    textAlign: "center",
                    marginTop: "12px",
                    fontSize: "13px",
                    color: "#6c757d",
                  }}
                >
                  Voice features are not supported in your browser. You can still type messages.
                </div>
              )}
            </div>
          </div>
        </div>
      </div>
    </div>
  )
}
