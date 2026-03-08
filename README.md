# ChattyBuddy
This project proposes an intelligent, child-friendly toy with a built‑in Arabic large language model (LLM) that can interact naturally with kids through conversation, stories, and games. The toy acts as a personal companion, answering questions, encouraging curiosity in a safe, age‑appropriate way. 


# Initial workflow:


## 1. Transcribe speech (Speech to Text) :
- An Arabic speech-to-text model is used to recognize Arabic speech and convert it into text
  
## 2. Enter the text into an Arabic LLM
- [HUMAIN ?](https://www.humain.com/en/humain-one/agents)
- The transcribed text is sent to a large language model such as ChatGPT, Gemini, Llama, or any other available model.

## 3. generate response and ensure its safe for children
Do we need to fine tune the model to make it child friendly, or can this be done through prompt engineering??

## 4. text to speach (stream response)
- The LLM’s text response is then passed to an Arabic text-to-speech model, which converts the generated Arabic text into audio.



### helpful sources?:
- -> [be-more-agent](https://github.com/brenpoly/be-more-agent)
- -> [arabic LLMs benchmarks](https://github.com/tiiuae/Arabic-LLM-Benchmarks)
