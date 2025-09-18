# Generative AI Quick Study Guide for Interviews

## 1. Basics of Generative AI
- **Definition**: AI models that generate new data (text, images, audio, code).  
- **Difference from discriminative AI**:  
  - Discriminative: predicts labels or classifies data.  
  - Generative: creates new samples/data.  
- **Key types**:
  - **Language Models (LLMs)**: GPT, LLaMA.  
  - **Diffusion Models**: DALL·E, Stable Diffusion.  
  - **GANs (Generative Adversarial Networks)**: Generator + Discriminator.  
  - **VAEs (Variational Autoencoders)**: probabilistic latent space generation.  

---

## 2. Core Concepts

### Transformers
- Attention mechanism: **self-attention** computes relationships between tokens.  
- Encoder-decoder vs decoder-only models.  

### Prompt Engineering
- Zero-shot, few-shot, chain-of-thought prompting.  
- Prompt templates to guide model output.  

### Fine-tuning
- **LoRA (Low-Rank Adaptation)**, **PEFT**.  
- **RLHF**: Reinforcement Learning from Human Feedback for alignment.  

### Embeddings
- Vector representation of text/images for similarity search.  
- Used in retrieval-augmented generation (RAG).  

### Inference Optimization
- Quantization (FP16, INT8) to reduce memory.  
- Model distillation for smaller models.  

---

## 3. Key Algorithms & Models
- **GPT series**: autoregressive LLMs.  
- **BERT**: masked language modeling, encoder-only.  
- **Diffusion models**: iterative denoising to generate images.  
- **GANs**: generator learns to fool discriminator.  

---

## 4. Applications
- **Text**: summarization, Q&A, chatbots.  
- **Images**: text-to-image generation (DALL·E, Stable Diffusion).  
- **Audio**: TTS, voice cloning.  
- **Code**: code generation (Copilot).  
- **Multimodal**: combining text, image, audio inputs.  

---

## 5. Evaluation
- **NLP metrics**: BLEU, ROUGE, perplexity.  
- **Generative metrics**: FID (images), Inception Score.  
- Human evaluation critical for generative quality.  
- Issues: hallucination, bias, factuality.  

---

## 6. Tools & Frameworks
- **Hugging Face Transformers**: pre-trained models + fine-tuning.  
- **LangChain / LlamaIndex**: model orchestration & RAG.  
- **OpenAI API / Anthropic Claude / Gemini**: cloud LLMs.  
- **PyTorch / TensorFlow**: training and experimentation.  

---

## 7. Challenges & Ethics
- **Hallucinations**: models generating false content.  
- **Bias & fairness**: datasets may propagate biases.  
- **Copyright & privacy**: generated content and training data.  
- **Safety & alignment**: preventing harmful outputs.  

---

## 8. Interview Q&A

**Q1:** Difference between discriminative vs generative models?  
**A1:** Discriminative predicts labels; generative creates new samples/data.  

**Q2:** How do transformers use self-attention?  
**A2:** Each token attends to every other token, producing contextual embeddings.  

**Q3:** What is RLHF and why is it important?  
**A3:** Reinforcement Learning from Human Feedback aligns model outputs with human preferences.  

**Q4:** How do diffusion models generate images?  
**A4:** Start from noise and iteratively denoise guided by a learned model.  

**Q5:** How do you optimize LLM inference in production?  
**A5:** Quantization, model distillation, batching, caching embeddings, and parallelization.  

**Q6:** Ethical concerns with generative AI?  
**A6:** Hallucinations, bias, privacy, copyright, misuse for deepfakes or misinformation.  

---

# ✅ Quick Tips
- Focus on understanding **transformers and LLMs fundamentals**.  
- Know **GANs, VAEs, diffusion models** basics.  
- Be prepared to discuss **prompting, fine-tuning, inference optimization**.  
- Be aware of **real-world applications and ethical considerations**.  
