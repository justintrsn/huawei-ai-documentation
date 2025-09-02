# 🧠 LLM Fundamentals: Learning What LLMs Are and Immediately Regretting It

Navigate back to [🏠 Home](../) | Skip to [🤖 AI Agents](../agents/) if you're brave

---

## Available Guides

| Guide | Description | Time to Understand | Existential Crisis Level |
|-------|-------------|-------------------|-------------------------|
| [🎓 LLM Basics](./llm-basics.md) | LLMs for Dummies (You're Not Dumb, This Stuff Is Just Weird) | 2 hours | ⭐⭐⭐ |
| [🪙 Tokens & Poverty](./tokens-and-poverty.md) | Why Your AI Bill Is So High | 1 hour | ⭐⭐⭐⭐ |
| [🌡️ Temperature & Top-P](./parameters.md) | The Drunk/Sober Slider for AI | 30 mins | ⭐⭐ |
| [🪟 Context Windows](./context-windows.md) | Why AI Forgets Everything Like That Friend Who "Forgot" They Owe You Money | 1 hour | ⭐⭐⭐⭐⭐ |

---

## 🎯 Learning Path

### If You're Completely New
1. Start with **LLM Basics** (prepare for confusion)
2. Learn about **Tokens** (prepare for poverty)
3. Understand **Temperature** (the fun knobs)
4. Grasp **Context Windows** (why everything breaks)

### If You're Building Something
1. Skip to **Context Windows** (you'll hit this limit first)
2. Read **Tokens & Poverty** (understand the bills)
3. Skim the others (you'll learn by breaking things)

### If You're Debugging Production
- Go directly to **Context Windows** (it's always context windows)
- Then check **Temperature** (someone set it to 2.0, didn't they?)

---

## 🤔 What You'll Learn

After reading this section, you'll understand:
- ✅ Why AI gives different answers to the same question
- ✅ Why your API bills are astronomical
- ✅ Why the AI forgot what you said 5 messages ago
- ✅ Why "just make it work" isn't a temperature setting
- ❌ Why we're doing this to ourselves (nobody knows)

---

## 💡 Quick Concepts

### The Lies They Tell You
- "LLMs understand language" → They're statistical parrots
- "Just increase context" → Your wallet cries
- "Temperature 0 is deterministic" → Mostly true, until it isn't
- "More tokens = better" → More tokens = more broke

### The Truth Nobody Mentions
- Context windows are never big enough
- Token counts are always higher than expected
- Temperature 0.7 is not always the answer
- The model will hallucinate with confidence

---

## 🚨 Common Gotchas

| Problem | You Think | Reality |
|---------|----------|---------|
| AI gives random answers | "It's broken" | Temperature is set to 1.5 |
| AI forgets everything | "Need better prompts" | Hit context window limit |
| Costs $500/day | "Normal usage" | Someone's sending images as base64 |
| Same prompt, different results | "Non-deterministic" | Temperature > 0 |

---

## 📊 The Reality Check

**What you expect after learning fundamentals:**
- 🎯 Perfect understanding of LLMs
- 💰 Optimized token usage
- 🎮 Complete control over outputs
- 🧘 Inner peace

**What you actually get:**
- 😵 More questions than answers
- 💸 Slightly lower bills (maybe)
- 🎲 Educated guessing at parameters
- 🤷 "It depends" as your catchphrase

---

## 🔗 Where to Go Next

**Ready for more pain?**
- [📝 Enhancements](../enhancements/) - Making LLMs slightly less stupid
- [🤖 AI Agents](../agents/) - Giving LLMs tools (what could go wrong?)
- [🚢 Deployment](../huawei/deployment-guides/) - Taking this mess to production

**Need immediate help?**
- [🚨 Emergency](../emergency/) - When the tokens are too damn high

---

## 📝 Notes from the Trenches

> "After learning about context windows, I finally understood why the AI kept forgetting our conversation. Then I checked the price of extending the context. Now I understand why we can't have nice things."
> 
> *- Anonymous Engineer, now farming*

> "Temperature 0.7 worked in dev. Temperature 0.7 broke in prod. Temperature 0.3 made it boring. Temperature 1.2 made it creative. Creative meant it told users to eat glass. We're back to 0.7."
> 
> *- Senior AI Engineer, questioning life choices*

---

## ⚠️ Final Warning

Understanding LLM fundamentals won't make you an AI expert. It will, however, make you understand why the AI experts look so tired all the time.

---

*Last updated: When token prices made us cry | Next update: When context windows get bigger (and more expensive)*