---
theme: academic
title: Large Language Model is All You Need
info: |
  ## Large Language Model is All You Need
  ### by Mohan Qiao
  Design Lab @ Concordia
class: text-center
drawings:
  persist: false
transition: fade-out
mdc: true
coverDate: '' 
---

<div class="flex flex-col items-center justify-center h-full">
  <h1 class="text-center">
    <span class="typing-container">
      <span ref="typeTarget" class="changing-word"></span><span ref="typingCursorRef" class="typing-cursor">|</span>
      <span class="static-text">&nbsp;is All You Need</span> </span>
  </h1>
  <div v-if="showPresenterDetails" class="presenter-info text-center mt-8 opacity-0 animate-fade-in-up animation-delay-500">
    <p class="text-2xl font-semibold">{{ presenterName }}</p>
    <p class="text-xl">{{ presenterAffiliation }}</p>
    <p class="text-lg">{{ currentDate }}</p>
  </div>
</div>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const typeTarget = ref(null);
const typingCursorRef = ref(null);

const originalWords = ['Attention', 'Imagination', 'Knowledge', 'Luck', 'Humor', 'Patience'];
const wordsToCycle = ref([...originalWords]);
let wordListIndex = ref(0);
let characterIndex = ref(0);
let isCurrentlyDeleting = ref(false);
let typingTimer;

// State variables for the new behavior
const attentionModeActivated = ref(false); // True after first interaction
const attentionIsTyped = ref(false);       // True once "Attention" is fully typed
const showPresenterDetails = ref(false);  // True to display presenter info

// Presenter details (you can customize these)
const presenterName = ref('Mohan Qiao');
const presenterAffiliation = ref('Design Lab @ Concordia');
const currentDate = ref(new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' }));

function masterTypeEffect() {
  if (!typeTarget.value || attentionIsTyped.value) {
    if (attentionIsTyped.value && typingCursorRef.value) {
      typingCursorRef.value.style.display = 'none'; // Hide cursor permanently
    }
    return; // Stop if Attention is typed or target is gone
  }

  let wordToProcess = wordsToCycle.value[wordListIndex.value];
  let typingSpeed = isCurrentlyDeleting.value ? 50 : 100; // Default speeds
  let delayAfterAction = 500; // Default delay before next word/action

  if (attentionModeActivated.value && !attentionIsTyped.value) {
    // --- ATTENTION SEQUENCE ---
    if (isCurrentlyDeleting.value) {
      // Deleting the word that was present when interaction occurred
      if (typeTarget.value.textContent.length > 0) {
        typeTarget.value.textContent = typeTarget.value.textContent.slice(0, -1);
      } else { // Deletion complete
        isCurrentlyDeleting.value = false;
        characterIndex.value = 0; // Reset for typing "Attention"
        typeTarget.value.style.color = 'red'; // Set color to red for "Attention"
        // wordsToCycle is not strictly needed here as we hardcode "Attention"
        delayAfterAction = 500; // Pause before typing "Attention"
      }
    } else {
      // Typing "Attention"
      wordToProcess = 'Large Language Model'; // Force word to "Attention"
      typingSpeed = 120; // Slightly slower, more deliberate typing for "Attention"
      
      if (characterIndex.value < wordToProcess.length) {
        typeTarget.value.textContent += wordToProcess[characterIndex.value];
        characterIndex.value++;
      } else { // "Attention" is fully typed
        attentionIsTyped.value = true;
        showPresenterDetails.value = true; // Trigger display of presenter info
        // Cursor hiding is handled at the top of this function if attentionIsTyped is true.
        clearTimeout(typingTimer); // Stop all further typing calls
        return;
      }
    }
  } else {
    // --- NORMAL WORD CYCLING (Initial Animation) ---
    if (isCurrentlyDeleting.value) {
      // Deleting current word
      if (characterIndex.value > 0) {
        typeTarget.value.textContent = wordToProcess.substring(0, characterIndex.value - 1);
        characterIndex.value--;
      } else { // Word fully deleted
        isCurrentlyDeleting.value = false;
        wordListIndex.value = (wordListIndex.value + 1) % wordsToCycle.value.length;
        // characterIndex is already 0, ready for typing next word
        delayAfterAction = 500; // Pause before typing new word
      }
    } else {
      // Typing current word
      if (characterIndex.value < wordToProcess.length) {
        typeTarget.value.textContent = wordToProcess.substring(0, characterIndex.value + 1);
        characterIndex.value++;
      } else { // Word fully typed
        isCurrentlyDeleting.value = true;
        // characterIndex is at wordToProcess.length, ready for deletion from end
        delayAfterAction = 1000; // Longer pause after full word, before deleting
      }
    }
  }

  // Schedule the next step of the effect
  clearTimeout(typingTimer);
  // Use delayAfterAction if a word was just completed/deleted, otherwise use typingSpeed
  const nextDelay = (isCurrentlyDeleting.value && characterIndex.value === 0) || 
                    (!isCurrentlyDeleting.value && characterIndex.value === wordToProcess.length && !attentionModeActivated.value) || // finished typing normal word
                    (attentionModeActivated.value && isCurrentlyDeleting.value && typeTarget.value.textContent.length === 0) || // finished deleting for attention
                    (attentionModeActivated.value && !isCurrentlyDeleting.value && characterIndex.value === 0 && typeTarget.value.textContent === '') // about to type attention
                    ? delayAfterAction : typingSpeed;
  typingTimer = setTimeout(masterTypeEffect, nextDelay);
}

const onSlideKeyPress = (e) => {
  // Listen for Right Arrow or Spacebar, but only if attention mode hasn't been activated yet
  if ((e.key === 'B' || e.key === ' ') && !attentionModeActivated.value) {
    e.preventDefault(); // Prevent default browser action (e.g., scrolling with space)
    
    attentionModeActivated.value = true; // Activate attention mode
    clearTimeout(typingTimer); // Stop any ongoing typing animation immediately

    isCurrentlyDeleting.value = true; // Set mode to deleting
    // characterIndex will be implicitly handled by typeTarget.value.textContent.length in the attention sequence
    
    // If typeTarget is empty (e.g. animation was between words), skip deletion phase for Attention
    if (typeTarget.value.textContent.length === 0) {
        isCurrentlyDeleting.value = false; // No need to delete
        characterIndex.value = 0;
        typeTarget.value.style.color = 'red';
    }

    masterTypeEffect(); // Call to start the deletion, then "Attention" typing sequence
  }
};

onMounted(() => {
  // Initial setup for the animation to start
  if (wordsToCycle.value.length > 0) {
    characterIndex.value = 0;
    isCurrentlyDeleting.value = false;
    if (typeTarget.value) {
      typeTarget.value.textContent = ''; // Ensure it starts blank
    }
  }
  typingTimer = setTimeout(masterTypeEffect, 500); // Start the initial animation after a brief delay

  window.addEventListener('keydown', onSlideKeyPress);
});

onUnmounted(() => {
  clearTimeout(typingTimer); // Clean up timer
  window.removeEventListener('keydown', onSlideKeyPress); // Clean up event listener
});
</script>

<style>
.typing-container {
  display: inline-block;
  position: relative;
}

.changing-word {
  display: inline-block;
  margin-right: 0; /* Adjusted if static text has space */
}

.static-text {
  display: inline-block;
}

.typing-cursor {
  display: inline-block;
  animation: blink 1s infinite;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

.presenter-info {
  /* Basic styling for presenter info, can be expanded */
  color: #333; /* Example color, adjust to your theme */
}

/* Simple fade-in animation for presenter details */
@keyframes fade-in-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
.animate-fade-in-up {
  animation: fade-in-up 0.5s ease-out forwards;
}
.animation-delay-500 {
  animation-delay: 0.2s; /* Small delay for presenter info to appear */
}

</style>
---
title: Table of Contents
---
# Table of Contents

<Toc text-sm minDepth="1" maxDepth="1" />

---
title: "The Spark: Attention is All You Need"
level: 1
---

# The Evolution of Understanding Language

<div style="text-align: justify;">

- **Early Attempts:** Computers read word-by-word.
    - *Challenge:* Forgetting the start of long sentences.
    - *Models:* RNNs & LSTMs showed promise but had limitations with long texts and training speed.

<div v-click="1">

- **The Breakthrough (2017):**
    - Paper: "**Attention Is All You Need<sup>1</sup>**" by Vaswani et al. (Google)
    - Introduced: The **Transformer** architecture.
    - **Key Mechanism:** Relied *entirely* on <span style="color: red;">**Attention**</span>.

</div>
<div v-click="2">

- **Paving the Way for LLMs:**
    - The Transformer's <span v-mark.red="3">parallel processing</span> and attention mechanism became a foundational building block for the first Large Language Models (LLMs), enabling significant advancements in understanding and generating human-like text.

</div>
</div>

<Footnotes separator>
  <Footnote :number=1><a href="https://arxiv.org/abs/1706.03762" rel="noreferrer" target="_blank">Attention is all you need. In <i>Advances in neural information processing systems (pp. 5998-6008)</i>.</a></Footnote>
</Footnotes>

---
title: "What is Attention? The Importance of Context"
level: 2
---

# What is Attention? <span class="text-sm opacity-75">(First, let's consider context)</span>

<div class="grid grid-cols-2 gap-6 pt-4">

<div v-click="1">
<h3 class="text-xl font-semibold mb-2">"Model" at MIT?</h3>
<div class="p-4 bg-gray-100 rounded-lg text-sm prose-sm" style="text-align: justify;">
At MIT, the term "model" can refer to various representations used across different fields. These include mathematical or computational models that simulate real-world systems, machine learning models trained on data to make predictions, physical models that replicate structures for study or testing, theoretical models that provide conceptual frameworks for understanding phenomena, and design models used in fields like architecture or product design.
</div>
</div>

<div v-click="2">
<h3 class="text-xl font-semibold mb-2">"Model" in Paris?</h3>
<div class="p-4 bg-gray-100 rounded-lg text-sm prose-sm" style="text-align: justify;">
A model in Paris refers to someone who works in the fashion industry, typically showcasing clothing, accessories, and products through runway shows, photo shoots, and advertisements. Paris, known as the fashion capital of the world, is home to top modeling agencies and events like Paris Fashion Week. Models in Paris can specialize in various areas, including runway, editorial, commercial, and fit modeling, all contributing to the global fashion scene.
</div>
</div>

</div>

<div v-click="3" class="mt-8 text-center text-xl">
<p>Clearly, words derive meaning from their <span v-mark.circle.orange="4"> context </span>.</p>
<p class="mt-2">The term <strong style="color: red;">"Attention"</strong> in AI is similar – it has a specific technical meaning we'll explore.</p>
</div>


---
title: "Attention: Understanding Nuance"
level: 2
---
# Attention: Understanding Nuance
<div class="pt-4" style="text-align: justify;">

<div v-click="1"> <div v-click="2"> 
    <p class="mt-2 text-lg">
      Just like you focus on <strong>key words</strong> to understand a sentence, <strong style="color: red;">"Attention"</strong> gives models a similar ability. Think of it as a <strong>smart highlighter</strong> that helps the AI pinpoint crucial information and relationships between words.
    </p>
  </div>

  <div v-click="3" class="mt-3 pl-4 text-md"> 
    <p class="font-semibold">This is key for tricky sentences. Consider this classic line from Groucho Marx<sup>1</sup>:</p>
    <blockquote class="my-2 p-3 bg-amber-100 rounded text-center italic text-gray-800 text-lg shadow">
      "One morning I shot an elephant <strong style="color: #b30000;">in my pajamas</strong>. How he got in my pajamas, I don't know."
    </blockquote>

   <div v-click="4"> 
      <p class="mt-1">
        The first part, <em class="text-gray-700">"I shot an elephant in my pajamas,"</em> presents an amusing ambiguity: Who's wearing the pajamas – me, or the elephant?
      </p>
      <p class="mt-2">
        AI "Attention" helps the model determine the more plausible meaning for that initial clause. By analyzing patterns learned from vast amounts of text, it "pays more attention" to the typically stronger and more logical connection between <strong style="color: #0077cc;">"I"</strong> (the shooter) and the descriptive phrase <strong style="color: #b30000;">"in my pajamas."</strong>
      </p>
   </div>
  </div>
</div>

</div>

<Footnotes separator>
  <Footnote :number=1>Line delivered by Groucho Marx (as Captain Spaulding) in the film <i>Animal Crackers</i> (Paramount Pictures, 1930).</Footnote>
</Footnotes>

---
title: "Generative AI & Large Language Models"
level: 1
---

# Generative AI & Large Language Models

<div class="pt-4" style="text-align: justify;">

So far, we've seen how "Attention" helps models *understand* language. Now, let's explore how Large Language Models (LLMs) use this understanding to *create* new text, making them powerful **Generative AI** tools.

What exactly makes an LLM "generative"? It's their ability to produce novel content—text, in this case—that is coherent, contextually relevant, and often indistinguishable from human-written text.

This generative capability is not magic; it's rooted in specific techniques.

</div>

---
title: "The Engine of Creation: Auto-Regressive Decoding"
level: 2
---

# The Engine of Creation: What Makes an LLM "Generative"?

<div class="pt-4" style="text-align: justify;">

At the heart of an LLM's ability to generate text is a process called **auto-regressive decoding**.

<div v-click="1" class="mt-4">

**1. Auto-Regressive Decoding:**

* **"Auto-regressive"** means the model predicts the *next* word (or token) in a sequence based on the words it has *already generated* in the current sequence.
* It's like building a sentence one word at a time, where each new word depends on the words that came before it.

    <div class="my-3 p-3 bg-sky-100 rounded text-center italic text-gray-800 text-lg shadow">
        Given an input like "The cat sat on the...", the model predicts the next most probable word (e.g., "mat"). Then, using "The cat sat on the mat...", it predicts the subsequent word...
    </div>
* This step-by-step generation allows LLMs to create extended, coherent passages of text.

</div>
</div>

---
title: "Guiding Generation: Prompting vs. Fine-Tuning"
level: 2
---

# Guiding Generation: Prompting vs. Fine-Tuning

<div class="pt-4" style="text-align: justify;">

<div class="grid grid-cols-1 md:grid-cols-2 gap-6 mt-4">

<div v-click="1" class="p-4 bg-teal-50 rounded-lg shadow">
<h3 class="font-semibold text-xl mb-2 text-teal-700">1. Few/Zero-Shot <span v-mark.circle.orange="3">Prompting</span></h3>
<ul class="list-disc pl-5 space-y-1 text-sm">
  <li>
    <strong>Zero-Shot:</strong> Asking the LLM to perform a task with no prior examples given in the prompt.
    <br/><em>e.g., "Summarize this article: [article text]"</em>
  </li>
  <li>
    <strong>Few-Shot:</strong> Guiding the LLM by providing a few examples of the task directly in the prompt.
    <br/><em>e.g., "Translate English to French: <br/> sea otter => loutre de mer <br/> peppermint => menthe poivrée <br/>  What is 'hello world' in French?"</em>
  </li>
  <li>
    <span class="font-semibold">Key Idea:</span> Relies on the LLM's pre-trained knowledge; model weights are not altered.
  </li>
</ul>
</div>

<div v-click="2" class="p-4 bg-orange-50 rounded-lg shadow">
<h3 class="font-semibold text-xl mb-2 text-orange-700">2. <span v-mark.circle.orange="4">Fine-Tuning</span></h3>
<ul class="list-disc pl-5 space-y-1 text-sm">
  <li>
    Involves further training a pre-trained LLM on a smaller, task-specific dataset.
  </li>
  <li>
    This process adjusts the model's internal weights to specialize it for that particular task or domain.
    <br/><em>e.g., Training a general LLM on thousands of medical research papers to make it an expert in medical Q&A.</em>
  </li>
  <li>
    <span class="font-semibold">Key Idea:</span> Adapts the model's knowledge and behavior, making it more proficient for specific applications. More resource-intensive than prompting.
  </li>
</ul>
</div>
</div>
</div>


---
title: "Understanding the Memory - Context Window"
level: 1
---

# Understanding the "Memory": Context Window

<div class="pt-4" style="text-align: justify;">

<div v-click="1">
<h3 class="text-xl font-semibold mb-2">What is a Context Window?</h3>
<p class="text-lg">
The <strong>context window</strong> (or "context length") of a Large Language Model (LLM) refers to the amount of text the model can consider or "remember" at any single point in time when processing input and generating a response. Think of it as the LLM's short-term or working memory.
</p>
</div>

<div v-click="2" class="mt-6">
<img 
  src="/assets/context-window-thinking.svg" 
  alt="Context Window Diagram" 
  style="display: block; margin: 0 auto; max-width: 55%; width: 600px;" 
  class="rounded-xl shadow-lg" 
/>
</div>
</div>

---
title: "LLM Context Windows"
level: 2
---

# LLM Context Windows

| Provider / Model Series | Representative Model(s)         | Max Announced Context Window (Tokens) |
|-------------------------|---------------------------------|---------------------------------------|
| OpenAI                  | GPT-4o series (incl. mini/nano) | 1M                                    | 
| Meta AI                 | Llama 4 Scout                   | 10M                                   |
| Anthropic               | Claude 3.7 Sonnet               | 200k                                  |
| Google                  | Gemini 2.5 Pro                  | 1M (with 2M coming soon)              | 
| xAI                     | Grok-3                          | 1M                                    | 
| DeepSeek AI             | DeepSeek-V3 / R1                | 128k                                  |


---
title: "How Big is That? Context Window Intuition"
level: 2
---

# How Big is That? Context Window Intuition

Let's get a feel for what those large token numbers mean in practice.
*(Rule of thumb: 1 token ≈ ¾ word in English)*

<div v-click="1">

* **~128k - 200k Tokens:**
  * Think of a very large book (~400-500 pages), like **"Moby Dick"** or **"War and Peace" (one volume)**.

</div>

<div v-click="2">

* **1 Million Tokens:**
  * Equivalent to **several novels** (~1,500-2,000 pages).
  * Imagine processing the **entire "Lord of the Rings" trilogy** or the **complete works of Shakespeare** at once.

</div>

<div v-click="3">

* **10 Million Tokens:**
  * About 7.5 million words.
  * Capable of holding **massive codebases** or **huge collections of research papers** simultaneously.

</div>

---
title: "Why Does Context *Feel* Shorter Than Advertised?"
level: 2
---

# Why Does Context *Feel* Shorter Than Advertised?

Even with huge context windows (like 1M+ tokens!), using them sometimes feels like the model's memory isn't quite that long. Here's why:

<div v-click="1">

* **The "Lost in the Middle" Problem:** Models often pay less attention to information buried deep in the middle of a long text<sup>1</sup>. They tend to recall info from the <span v-mark.red="2">beginning</span> and <span v-mark.red="3">end</span> much better. This is sometimes called a U-shaped attention bias.

</div>

<div v-click="4">

* **Effective vs. Theoretical Length:** The *maximum* tokens a model *can* handle (theoretical limit) isn't the same as the amount it can *effectively use* without performance dropping. Performance on complex tasks can decrease significantly well before the absolute maximum is reached.

</div>


<div v-click="5">

* **How We Interact:** Chat interfaces might use tricks like summarizing older parts of the conversation or retrieving relevant snippets (like Retrieval-Augmented Generation or RAG) instead of constantly feeding the *entire* history into the model's raw context window. 

</div>

<Footnotes class="text-xs">
  <Footnote number=1><a href="https://aclanthology.org/2024.findings-acl.890/" target="_blank">Found in the middle: Calibrating Positional Attention Bias Improves Long Context Utilization - ACL Anthology</a></Footnote>
</Footnotes>

---
title: "Best Practice: Single Responsibility"
level: 2
---

# Best Practice: Single Responsibility Principle (SRP)

<div v-click="1">

An object (here: a <span style="color: red;">context</span>) should only **do one thing, and do it well.** Each piece should have a single, well-defined job or responsibility<sup>[1]</sup>.

</div>

<span v-click="2"> Example: </span>
<blockquote v-click="3" class="my-2 p-3 bg-amber-100 rounded text-center italic text-gray-800 text-lg shadow">
      Build an LLM Research Assistant
</blockquote>

<span v-click="4">

**This is an excellent chance to apply EBD!**

</span>

<Footnotes class="text-xs">
  <Footnote number=1>Martin, Robert C. (2018). <a href="https://www.pearson.com/en-us/subject-catalog/p/clean-architecture-a-craftsmans-guide-to-software-structure-and-design/P200000009528/9780134494326" target="_blank"><em>Clean architecture : a craftsman's guide to software structure and design</em></a>. Boston. ISBN 978-0-13-449432-6. OCLC 1003645626</Footnote>
</Footnotes>

---
title: "Single Responsibility (Applied to Context)"
level: 3
---

# Single Responsibility (Applied to Context)

<div class="grid grid-cols-2 gap-x-4 mt-4">

<div v-click="1" class="p-2 bg-red-50 rounded-lg shadow-sm">
  <h3 class="font-semibold text-base mb-2 text-red-700">❌ Overloaded Context (Violating SRP)</h3>
  <p class="text-xs mb-1">Trying to make a <strong>single prompt/context</strong> handle too many distinct jobs:</p>
  <ul class="list-disc pl-4 space-y-0.5 text-xs">
    <li>"Here's a keyword: find papers."</li>
    <li>"Now parse these PDFs found."</li>
    <li>"Summarize the parsed text."</li>
    <li>"Based on the summary, suggest research questions."</li>
    <li>"Also, draft a lit review section using all the above."</li>
  </ul>
  <p class="mt-2 text-xs italic text-red-600">*Mixing unrelated tasks makes the context confusing, hard to manage, and likely reduces LLM performance.*</p>
</div>

<div v-click="2" class="p-2 bg-green-50 rounded-lg shadow-sm">
  <h3 class="font-semibold text-base mb-2 text-green-700">✅ Focused Contexts (Applying SRP)</h3>
  <p class="text-xs mb-1">Use separate, focused contexts/prompts for each agent's single responsibility:</p>
  <ul class="list-disc pl-4 space-y-0.5 text-xs">
    <li><strong><code>Fetcher Context</code></strong>: Input: Keywords -> Output: Paper list/files.</li>
    <li><strong><code>Summarizer Context</code></strong>: Input: Paper text -> Output: Summary.</li>
    <li><strong><code>Extractor Context</code></strong>: Input: Paper text -> Output: Key entities.</li>
    <li><strong><code>QuestionGen Context</code></strong>: Input: Summary/Entities -> Output: Questions.</li>
    <li><strong><code>Drafter Context</code></strong>: Input: Outline/Sources -> Output: Draft text.</li>
  </ul>
  <p class="mt-2 text-xs italic text-green-600">*Each context is clear, manageable, easier to optimize, and targets a specific LLM capability.*</p>
</div>

</div>

---
title: "Which LLM Should I Use? (The 'Elephants' in the Room)"
level: 1
---

# Which LLM Should I Use? (The 'Elephants' in the Room)

<div v-click="1">

Choosing the right Large Language Model (LLM) feels complex because... well, it is! There's a rapidly growing zoo of options out there.

</div>

<div v-click="2">

**The Landscape Changes *Fast***

* **The "Elephants":** We constantly hear about models from OpenAI (GPT series), Google (Gemini series), Anthropic (Claude series), Meta (Llama series), xAI (Grok), Mistral, Cohere, DeepSeek, and many others.
* **Constant Flux:** New models, major updates, and performance benchmarks are released seemingly **every few weeks or months**.

<span v-click="3"> 

**Tips:** Keep an eye on the benchmarks.

</span>
</div>

---
layout: figure
figureCaption: Artificial Analysis Intelligence Index
figureFootnoteNumber: 1
figureUrl: "assets/artificial_analysis.png"
title: Benchmarks
level: 3
hideInToc: true
---

# Artificial Analysis

<Footnotes separator>
  <Footnote :number=1><a href="https://artificialanalysis.ai/" rel="noreferrer" target="_blank">Artificial Analysis</a></Footnote>
</Footnotes>

---
layout: figure
figureCaption: LiveBench - A Challenging, Contamination-Free LLM Benchmark
figureFootnoteNumber: 1
figureUrl: "assets/livebench.png"
title: LiveBench
level: 3
---

# LiveBench

<Footnotes separator>
  <Footnote :number=1><a href="https://livebench.ai/#/" rel="noreferrer" target="_blank">LiveBench</a></Footnote>
</Footnotes>

---
layout: figure
figureCaption: Chatbot Arena
figureFootnoteNumber: 1
figureUrl: "assets/chatbot_arena.png"
title: Chatbot Arena
level: 3
---

# Chatbot Arena

<Footnotes separator>
  <Footnote :number=1><a href="https://lmarena.ai/?leaderboard" rel="noreferrer" target="_blank">Chatbot Arena</a></Footnote>
</Footnotes>

---
title: "Google"
level: 2
---

# Google LLMs

<div v-click="1">

Google's Gemini is a family of multimodal AI models, with key versions like **Gemini 2.5 Pro** (for top-tier performance and complex reasoning) and **Gemini 2.5 Flash** (optimized for speed and cost-efficiency). They are known for large context windows and native multimodality.

</div>

<div v-click="2">

**Price**: Include in Google AI Premium: CA$26.99 / month

</div>

<div v-click="3">

**Context Window**: <span v-mark.circle.red="6"> 1M tokens </span>

</div>

<div v-click="4">

**<span v-mark.circle.red="7"> Deep Research</span>** : Reason - <span v-mark.red="7">Google Search</span> - Browse

</div>

<div v-click="5">

<img
src="/assets/deepresearch_gemini_vs_openai.png"
alt="Gemini Deep Research Benchmark"
style="display: block; margin: 0 auto; max-width: 45%; width: 600px;"
class="rounded-xl shadow-lg"
/>

</div>

---
title: "OpenAI"
level: 2
---

# OpenAI LLMs

<div v-click="1">

OpenAI's GPT series, particularly **GPT-4o**, represents their flagship multimodal models. They are recognized for advanced reasoning, strong creative text generation, coding assistance, and the ability to process and generate text, audio, and images.

</div>

<div v-click="2">

**Price**: ChatGPT Plus: CA$28.01 / month (ChatGPT Pro: CA$280.12 / month)

</div>

<div v-click="3">

**Context Window**: <span v-mark.circle.red="6"> up to 1M tokens </span>

</div>

<div v-click="4">

**<span v-mark.circle.red="7"> DALL·E 3</span>**: Image Generation (local modification)

</div>

<div v-click="5">

<img
src="/assets/dalle.png"
alt="Conceptual image representing DALL·E image generation"
style="display: block; margin: 0 auto; max-width: 25%; width: 600px;"
class="rounded-xl shadow-lg"
/>

</div>

---
title: "Anthropic"
level: 2
---

# Anthropic LLMs

<div v-click="1">

Anthropic's Claude series, with models like **Claude 3.7 Sonnet**, **Opus**, and **Haiku**, emphasizes safety, helpfulness, and strong performance. They are known for robust reasoning, handling long contexts, and increasingly, sophisticated coding abilities, including an "extended thinking" mode in newer versions.

</div>

<div v-click="2">

**Price**: Claude Pro: CA$28.01  / month (Claude Max: CA$280.12 / month)

</div>

<div v-click="3">

**Context Window**: 200k tokens (for Claude 3.7 Sonnet, Claude 3.5 Sonnet/Haiku, Claude 3 Opus)

</div>

<div v-click="4">

**<span v-mark.circle.red="7"> Frontend Coding</span>**: UI Generation - <span v-mark.red="7">HTML/CSS/JS</span> - Complex Apps

</div>

<div v-click="5">

<img
  src="/assets/claude_frontend.png"
  alt="Conceptual image of Anthropic Claude assisting with frontend web development"
  style="display: block; margin: 0 auto; max-width: 33%; width: 600px;"
  class="rounded-xl shadow-lg"
/>

</div>

---
title: "xAI"
level: 2
---

# xAI LLMs

<div v-click="1">

xAI, founded by Elon Musk, aims to "understand the universe" with its AI models like **Grok**. Grok is designed to answer a wide range of questions, including "spicy" ones rejected by other AIs, and has a unique advantage with real-time access to information via the X platform. Grok-3 is a recent powerful iteration.

</div>

<div v-click="2">

**Price**: Super Grok: CA$30.00 / month (CA$300.00 / year)

</div>

<div v-click="3">

**Context Window**: <span v-mark.circle.red="6"> 1M tokens </span> (for Grok-3)

</div>

<div v-click="4">

**<span v-mark.circle.red="7"> Live X Data Feed</span>**: Real-time Insights - <span v-mark.red="7">X Platform</span> - Current Events

</div>

---
title: "Workflow Examples"
level: 1
---

# Workflow Example 1: Literature Review

<div v-click="1" class="mt-4">

**Goal:**
<p class="text-lg">Produce a thorough and well-written academic literature review.</p>

</div>

<div v-click="2" class="mt-6">

**Workflow:**
<ul class="list-disc pl-5 space-y-2">
  <li>
    <strong>Gemini</strong> (Deep Research + PDF Analysis):
    <ul class="list-circle pl-5 text-sm">
      <li>Ingest and deeply analyze a corpus of research papers (PDFs).</li>
      <li>Extract key themes, methodologies, findings, and citations.</li>
    </ul>
  </li>
  <li>
    <span class="text-2xl">➡️</span> <strong>ChatGPT</strong> (Summarization & Writing):
    <ul class="list-circle pl-5 text-sm">
      <li>Synthesize the extracted information into coherent summaries.</li>
      <li>Draft sections of the literature review with clear narrative flow.</li>
      <li>Refine language and ensure academic tone.</li>
    </ul>
  </li>
</ul>

</div>

<div v-click="3" class="mt-6">

**Outcome:**

<p class="text-lg text-green-600 font-semibold">A deeply researched, well-structured, and eloquently written literature review.</p>
</div>

---
title: "Workflow Example 2: Visual Report"
level: 2
---

# Workflow Example 2: Aesthetic Visual Report

<div v-click="1" class="mt-4">

**Goal:**
<p class="text-lg">Create an engaging, interactive report with strong visual storytelling from complex data.</p>
</div>

<div v-click="2" class="mt-6">

**Workflow:**
<ul class="list-disc pl-5 space-y-2">
  <li>
    <strong>Gemini</strong> (Deep Research & Data Synthesis):
    <ul class="list-circle pl-5 text-sm">
      <li>Gather, clean, and synthesize underlying data from various sources.</li>
      <li>Identify key insights and narratives within the data.</li>
    </ul>
  </li>
  <li>
    <span class="text-2xl">➡️</span> <strong>Claude</strong> (Visualization Design & Code):
    <ul class="list-circle pl-5 text-sm">
      <li>Design the structure and flow for an aesthetic visual presentation.</li>
      <li>Generate code (e.g., HTML/CSS/JavaScript with libraries like D3.js or Plotly.js) for interactive charts and dashboards.</li>
    </ul>
  </li>
</ul>
</div>

<div v-click="3" class="mt-6">

**Outcome:**
<p class="text-lg text-green-600 font-semibold">An insightful data report that is both visually appealing and interactively explorable.</p>
</div>

---
title: "Workflow Example 3: 'Vibe Coding'"
level: 2
---

# Workflow Example 3: "Vibe Coding"

<div v-click="1" class="mt-4">

**Goal:**
<p class="text-lg">Rapidly prototype and develop a software application with a good architectural foundation.</p>
</div>

<div v-click="2" class="mt-6">

**Workflow:**
<ul class="list-disc pl-5 space-y-2">
  <li>
    <strong>ChatGPT</strong> (Initial Ideation & User Stories):
    <ul class="list-circle pl-5 text-sm">
      <li>Brainstorm project ideas and define core features based on a general concept.</li>
      <li>Draft initial user stories and requirements.</li>
    </ul>
  </li>
  <li>
    <span class="text-2xl">➡️</span> <strong>Claude</strong> (Architecture & Core Logic):
    <ul class="list-circle pl-5 text-sm">
      <li>Design the high-level project structure and module responsibilities.</li>
      <li>Concrete code writing.</li>
    </ul>
  </li>
  <li>
    <span class="text-2xl">➡️</span> <strong>Gemini</strong> (Large-Scale Context & Inspection):
    <ul class="list-circle pl-5 text-sm">
      <li>Assist in debugging complex interactions, inspecting code quality, or understanding system-wide behavior.</li>
    </ul>
  </li>
</ul>
</div>
<p v-click="3" class="mt-6">

**Outcome:** <span v-click="3" class="text-lg text-green-600 font-semibold">Make real apps without needing to know how to code.</span>
</p>

---
title: "Tips"
level: 1
---

# Tip 1: Brainstorm, Don't Prescribe

<div v-click="1" class="mt-4">

**The Tip:**
<p class="text-lg">Do NOT be solution-oriented from the very start.</p>
</div>

<div v-click="2" class="mt-6">

**Why?**
<ul class="list-disc pl-5 space-y-1">
  <li>Jumping directly to what you <i>think</i> is the solution might make you miss better, more efficient, or more creative alternatives.</li>
  <li>LLMs can be powerful thought partners and idea generators if you let them.</li>
</ul>
</div>

<div v-click="3" class="mt-6">

**How to Apply It:**
<ul class="list-disc pl-5 space-y-1">
  <li>Always **brainstorm with an LLM** <em>before</em> you commit to a specific approach or tool for a project.</li>
  <li>Ask questions like:
    <ul class="list-circle pl-5 text-sm mt-1">
      <li>"What tools, frameworks, or methodologies are ideal for [describe your task/problem]?"</li>
      <li>"Help me refine my actual need here: I want to [initial thought], but what outcomes am I <i>really</i>> aiming for?"</li>
    </ul>
  </li>
</ul>
</div>

---
title: "Tip 2: Evaluate, Don't Just Create"
level: 2
---

# Tip 2: Evaluate, Don't Just Create

<div v-click="1" class="mt-4">

**The Tip:**
<p class="text-lg">

Your ability to **evaluate** an LLM's output is often more crucial (and initially easier) than asking it to perform a complex action perfectly from scratch.

</p>
</div>

<div v-click="2" class="mt-6">

**Why?**
<ul class="list-disc pl-5 space-y-1">
  <li>LLMs can generate vast amounts of content, but it won't always be accurate, complete, or perfectly aligned with your nuanced requirements on the first try.</li>
  <li>Your domain expertise and critical thinking are essential for quality control and refinement.</li>
</ul>
</div>

<div v-click="3" class="mt-6">

**How to Apply It:**
<ul class="list-disc pl-5 space-y-1">
  <li>Instead of: "Write a full research paper on [topic]."</li>
  <li>Try: "Here's an outline for a paper on [topic]. Can you critique it, suggest improvements, or identify gaps?" or "Generate three different introductions for this abstract."</li>
  <li>Use LLMs to generate options, drafts, or components. Then, you evaluate, select, combine, and guide the refinement process.</li>
  <li>It's often more efficient for an LLM to critique or compare existing material (or its own previous output) than to achieve perfection in one go for complex tasks.</li>
</ul>
</div>

---
layout: figure
figureCaption:
figureFootnoteNumber: 1
figureUrl: "assets/augmentation_vs_automation.png"
title: Augmentation or Automation?
level: 3
---

# Augmentation or Automation?

<Footnotes separator>
  <Footnote :number=1><a href="https://www.anthropic.com/news/the-anthropic-economic-index" rel="noreferrer" target="_blank">The Anthropic Economic Index</a></Footnote>
</Footnotes>

---
title: "Tip 3: Be the Project Manager"
level: 2
---

# Tip 3: Be the Project Manager

<div v-click="1" class="mt-4">

**The Tip:**
<p class="text-sm">

Act as the **project manager** or architect; let the LLMs be your highly skilled (but sometimes literal-minded) team members handling detailed execution.</p>
</div>

<div v-click="2" class="mt-6">

**Why?**
<ul class="list-disc pl-2 space-y-1 text-sm">
  <li>LLMs excel at executing well-defined, focused tasks when given clear instructions and context.</li>
  <li>You bring the strategic vision, overall project structure, quality standards, and integration know-how.</li>
</ul>
</div>

<div v-click="3" class="mt-6">

**How to Apply It:**
<ul class="list-disc pl-5 space-y-1 text-sm">
  <li>
    <strong>Decompose</strong>: Break down your complex project into smaller, manageable sub-tasks or modules.
  </li>
  <li>
    <strong>Delegate</strong>: Assign these specific sub-tasks to an LLM (e.g., "Write a Python function that does X, given input Y, producing output Z," or "Draft an introductory paragraph for a section on [specific subtopic], using these key points...").
  </li>
  <li>
    <strong>Oversee</strong>: You remain responsible for the high-level design, how components fit together, and the overall quality and coherence of the final product.
  </li>
</ul>
</div>

---
title: "Tip 4: Embrace Mutual Prompting"
level: 2
---

# Tip 4: Embrace Mutual Prompting

<div v-click="1" class="mt-4 text-sm">

**The Insight:**
<p class="text-lg text-sm">Humans also function somewhat like LLMs – we process information, look for patterns, and generate responses based on our "training" (experience & knowledge).</p>
</div>

<div v-click="2" class="mt-6 text-sm">

**The Shift:**
<p class="text-lg text-sm">

Move beyond simply *querying* information from an LLM. Engage in **mutual-prompting**: let the LLM instruct you, ask *you* questions, and guide *your* exploration.</p>
</div>

<div v-click="3" class="mt-6 text-sm">
  
**How to Apply It:**
<ul class="list-disc pl-5 space-y-1 text-sm">
  <li>Turn the tables – ask the LLM to act as a guide or Socratic questioner.</li>
  <li>Example Prompts (or use EBD!!!):
    <ul class="list-circle pl-5 text-sm mt-1">
      <li>"I need to understand [complex concept, e.g., 'quantum entanglement'] thoroughly. What key questions should I be asking myself to build a solid understanding?"</li>
      <li>"I'm planning a research project on [topic]. Can you act as an experienced supervisor and ask me critical questions about my methodology, assumptions, and potential challenges?"</li>
  </ul>
  </li>
</ul>
</div>

---
title: "Key Takeaways: Mastering LLMs"
level: 1
---

# Key Takeaways: Mastering LLMs

<div v-click="1" class="mt-4">

* **LLMs are All you need** 

</div>

<div v-click="2" class="mt-3">

* **Strategic Use is Key** 

</div>

<div v-click="3" class="mt-3">

* **Combine Strengths (Hybrid Workflows)** 
 
</div>

<div v-click="4" class="mt-3">

* **Be the Project Manager, Not Just the User**

</div>

<div v-click="5" class="mt-3">

* **Evaluate, Don't Just Passively Create** 

</div>


---
layout: center
title: "Thank You & Q&A"
hideOnToc: true
---

# Thank You

<br>
<br>

## Questions?

<br>
<br>
<br>

<div class="text-lg">
  <p>Mohan Qiao</p>
  <p>Design Lab @ Concordia</p>
  <p class="text-sm opacity-75">{{ new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' }) }}</p>
</div>

