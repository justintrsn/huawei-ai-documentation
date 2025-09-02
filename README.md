# 🎭 The AI Survival Guide for Huawei
## *Or: How to Pretend You Know What You're Doing When Everyone Who Actually Knows Has Left*

---
# Viewing
View this in VS Code and `Ctrl + Shift + V` for better experience

## A Letter to Future Survivors

Dear Future Huawei Employee,

If you're reading this, it means:
1. You've been asked to "work with AI"
2. The person who knew things has left (probably my mentor, possibly me)
3. You're googling "what is LLM" at 2 AM
4. You're wondering if it's too late to switch to a different project

First, breathe. Second, welcome to the club. Third, this guide exists because I've been there, and I refuse to let you suffer alone.

---

## Why This Guide Exists (A Confession)

Woo Yan Kit - the ONLY person who actually understood LLMs and AI agents in this team - is leaving. I'm writing this while frantically taking notes, asking stupid questions, and pretending I understand when they explain things for the 47th time.

This guide is my attempt to document everything before the knowledge walks out the door. It's written in a sarcastic tone because:
- Technical documentation is boring enough to cause actual brain damage
- Laughing at our pain makes it hurt less
- If we're going to suffer, we might as well smile about it
- Serious guides make me want to quit tech and do OnlyFriends

**The Sacred Rule**: All future contributions MUST maintain the sarcastic, self-deprecating tone. We're not just building AI systems; we're building a support group disguised as documentation.

---

## 📚 The Knowledge Tree (From "What's an LLM?" to "Kill Me Now")

### 🎓 **Level 0: Prerequisites**
*"Things you pretend to know but secretly Google"*
- [What Is Python and Why Does It Hate You](/prerequisites/python-pain.md)
- [APIs: Making Computers Talk to Each Other Badly](/prerequisites/api-basics.md)
- [JSON: The Format That's Almost But Not Quite Human-Readable](/prerequisites/json-suffering.md)

### 🧠 **Level 1: LLM Fundamentals**
*"Learning what LLMs are and immediately regretting it"*
- [LLMs for Dummies (You're Not Dumb, This Stuff Is Just Weird)](/fundamentals/llm-basics.md)
- [Tokens: Why Your AI Bill Is So High](/fundamentals/tokens-and-poverty.md)
- [Temperature and Top-P: The Drunk/Sober Slider](/fundamentals/parameters.md)
- [Context Windows: Why AI Forgets Everything](/fundamentals/context-windows.md)

### 🎯 **Level 2: Making LLMs Slightly Less Stupid**
*"Enhancement techniques that sometimes work"*
#### 📝 [Why Do I need enhancements](/enhancements/why-tf.md)
- Source: Trust me bro
- LLMs gaslighting you that the current president of Indonesia is Jokowi 
- What company data you were talking about again?

#### 📝 [Prompt Engineering: Talking to AI Like It's a Confused Toddler](/enhancements/prompt-engineering.md)
- Basic Prompting: "Please work. Pretty please?"
- Chain of Thought: Making AI show its homework
- Few-Shot Learning: Teaching by example (AI still won't learn)
- System Prompts: Setting expectations (AI will ignore them)

#### 📚 [RAG: Because LLMs Can't Remember Anything](/enhancements/rag.md)
- What is RAG: Giving AI a cheat sheet
- Vector Databases: Where embeddings go to die
- Chunking Strategies: Breaking things into smaller broken things
- Retrieval: Finding the needle in the haystack (there is no needle)

#### 🔧 [Fine-Tuning: Teaching Old Models New Tricks](/enhancements/fine-tuning.md)
- When to Fine-Tune: Almost never
- How to Fine-Tune: Painfully
- Why You Probably Shouldn't: But you will anyway
- Evaluating Results: "It's worse now"

### 🤖 **Level 3: AI Agents (Here Be Dragons)**
*"Making AI do actual work (theoretically)"*
- [Introduction to Agents: Chatbots with Delusions of Grandeur](/agents/introduction-to-agents.md)
- [MCP Protocol: The Standard Nobody Wanted But Everyone Needs](/agents/mcp-protocol.md)
- [Tool Calling: Giving AI Dangerous Capabilities](/agents/tool-calling.md)
- [Building Your First Agent: It Won't Work](/agents/building-your-first-agent.md)

### 🏢 **Level 4: Huawei-Specific Suffering**
*"Because every cloud provider must be special"*
- [MCP in Huawei Cloud: The Marriage Nobody Asked For](/huawei/mcp-in-huawei-cloud.md)

#### 🚢 **Deployment Guides: Step-by-Step Suffering**
*"Because deploying should totally be this complicated"*
- [01 - SWR Docker Push: Authentication Dance of Death](/huawei/deployment-guides/01-swr-docker-push.md)
- [02 - ECS Deployment: "It's Just Web Hosting"](/huawei/deployment-guides/02-ecs-deployment.md)

---

## 🗺️ Your Learning Path

### If You Know Nothing (Welcome!)
1. Start with Prerequisites (yes, even if you think you know Python)
2. Read LLM Fundamentals (twice, you won't get it the first time)
3. Try Prompt Engineering (it's the easiest enhancement)
4. Build a simple chatbot (it will be terrible)
5. Cry a little
6. Continue to RAG when you feel better

### If You're Replacing Someone Who Left
1. Read everything they wrote (if anything exists)
2. Read this guide's Huawei section thoroughly
3. Find their code (good luck)
4. Try to run it (it won't work)
5. Read the MCP and Agents section
6. Follow the deployment guides step by step
7. Rebuild everything from scratch
8. Document what you build (or doom your successor)

### If You're Already Experienced
1. Skip to the Huawei-specific section
2. Laugh at our pain
3. Fix our mistakes
4. Add your own sarcastic guides
5. Don't leave without documenting things (please)

### If You Need to Deploy Something Right Now
1. Go directly to [Deployment Guides](/huawei/deployment-guides/)
2. Start with [SWR Docker Push](/huawei/deployment-guides/01-swr-docker-push.md)
3. Follow the numbered sequence (don't skip steps, you'll regret it)
4. Come back and read the theory later
5. Document what broke and how you fixed it

---

## 🛠️ Repository Structure
*"Organization is the illusion of control"*

```
ai-survival-guide/
├── README.md                    # You are here (unfortunately)
├── prerequisites/               # Basic knowledge needed
├── fundamentals/               # LLM basics
├── enhancements/              # Making LLMs slightly less stupid
│   ├── prompt-engineering.md
│   ├── rag.md
│   ├── fine-tuning.md
|   └── why-tf.md
├── agents/                    # AI Agents 
├── huawei/                    # Huawei-specific suffering
│   ├── deployment-guides/     # Step-by-step deployment pain
│   │   ├── 01-swr-docker-push.md
│   │   ├── 02-ecs-deployment.md
│   └── [other huawei guides] # TODO in the future maybe IDK Im too lazy
└── emergency/               # When everything is on fire
```

---

## 🤝 Contributing to This Guide

### The Rules
1. **Maintain the sarcastic tone** - We suffer together, we laugh together
2. **Include real examples** - That actually failed before working
3. **Document the pain points** - Others need to know what hurt you
4. **Add your horror stories** - The worse, the better
5. **Keep it practical** - Theory is nice, but we need things to work
6. **Follow the deployment guide pattern** - Numbered, sequential, and brutally honest

### What We Need
- Your deployment disasters and victories
- Your prompt engineering failures and successes
- Your production nightmares and solutions
- Your Huawei-specific discoveries
- Translations of Chinese error messages
- More deployment guides (CCE, ModelArts, etc.)

### How to Contribute
```bash
# Fork this repo
# Add your painful wisdom
# Submit a PR with a sarcastic commit message
git commit -m "Added guide on why vector databases hate happiness"
```

---

## 🚨 Emergency Resources

### When Everything Is Broken
- [The Panic Guide: First Steps When Production Dies](/emergency/panic.md)
- [Who to Call: Contact List of People Who Might Help](/emergency/contacts.md)
- [Career Alternatives: Is Farming Really That Bad?](/emergency/career-change.md)

### Useful Links
- [Huawei Cloud Console](https://console.huaweicloud.com) - Where dreams go to die
- [OpenAI API Docs](https://platform.openai.com/docs) - Actually good documentation
- [Anthropic MCP](https://modelcontextprotocol.io) - The protocol you'll eventually use
- [Stack Overflow](https://stackoverflow.com) - Your real mentor
- [Twitter/X AI Community](https://twitter.com) - For moral support

---

## 🎬 Final Words

To whoever reads this:

You're about to embark on a journey that's fascinating but mostly frustrating. You'll build things that seem like magic, and you'll spend hours debugging things that make no sense. You'll feel like a genius one moment and an idiot the next.

This is normal.

The AI field moves so fast that everything you learn today will be obsolete tomorrow. The model that works perfectly in development will hallucinate in production. The prompt that generated perfect results will suddenly start returning recipes for banana bread.

This is also normal.

Remember:
- Document everything (your future self will thank you)
- It's okay to not understand everything
- The experts are also googling things
- If it works, don't touch it
- If it doesn't work, try adding "please" to your prompt
- When all else fails, blame the context window
- Follow the deployment guides in order (seriously, don't skip steps)

Keep the sarcasm alive. Keep documenting the pain. And most importantly, help the next person who stumbles into this chaos.

Good luck. You'll need it.

---

*P.S. - If you're my replacement: if you are wondering why. We don't know why. We've stopped asking. This is Huawei*

---

## 📝 Document History

| Version | Date | Author | Contribution |
|---------|------|--------|--------------|
| 1.0 | 2025-01-09 | Justin | [Woo Yan Kit](linkedin.com/in/yankit-woo?originalSubdomain=sg) - Initial structure and deployment guides |
| 1.1 | TBD | You? | Your pain points here |
| 1.2 | TBD | Someone | More suffering documented |

---

## 🏆 Hall of Fame (Survivors)

- [**Woo Yan Kit**](linkedin.com/in/yankit-woo?originalSubdomain=sg): Who knew things but left anyway
- **Justin**: Who wrote this while crying and fighting with SWR authentication
- **You**: For reading this far
- **Future You**: For not quitting (yet)

---

*"Why pass your probation or test if you can just pass away."*

*- Anonymous Huawei Employee, 3 AM, crying after writing*