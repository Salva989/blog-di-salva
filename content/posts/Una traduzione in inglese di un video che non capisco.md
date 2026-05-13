So, only a few days have passed since the release of DS4, this project I told you about, which today, by the way, I renamed to DARF Star 4 to avoid confusion with DeepSeek 4, while obviously keeping the initials because it’s an inference engine for DeepSeek 4. But I really like the dwarf star idea because this project is essentially trying to condense an enormous mass of weights into a very small space — we’re talking about 283 billion parameters, an enormous amount of, uh, intelligence from a near-frontier model packed into a fairly constrained space.

And here we are. A lot has happened. People have been talking about it everywhere, especially because the project got picked up in a tweet by the CEO of Y Combinator, so you can imagine — everyone started messaging me and so on.

By the way, I don’t want to sound ungrateful, but I wasn’t looking for this kind of attention, and honestly it’s always a bit difficult for me to handle it. Still, I tried to do my best.

So first of all, what happened today? Let’s start by looking at the features that were added to the project today, but at the end of this video I also want to reflect a bit on why a project like this generates so much interest and hype in such a short time.

For example, today even Giorgi Gerganov — the author of llama.cpp, I think that’s how you pronounce it, or maybe Gerganov, I’m not sure — commented on one of my issues related to 4-bit inference, because our engine supports both 2-bit and 4-bit inference. The Q4 quantization we use isn’t equivalent to FP4 even though both are technically 4-bit. One is a dynamic numerical representation format, while the other uses a more linear scaling, so there’s a difference.

Since GPT-OSS 120B was also released in this same format and llama.cpp performs inference directly in that format, I had opened this issue mostly for myself. What’s interesting is that he replied saying he plans to benchmark how faithful the 2-bit quantizations really are. And speaking of fidelity, I want to show you something.

Today I did a Tetris implementation session. This is a model that genuinely works locally — no joke. It implemented Tetris using SDL, because plain C would have been too easy, so I deliberately made it harder. It had to go through several edits, and here you can really see that this isn’t just about regurgitating a memorized implementation. At most it memorized the basic layout, but things get much more complex here.

Then I asked it to port the implementation to JavaScript while keeping the same C-style structure. Look at this — and also look at how many tokens were involved. Here’s the session: I had reached 82,000 tokens, and the model — which supports a 1 million token context window — still had absolutely no issues. Even with huge context windows it remains stable.

Now let me show you something really cool: disk KV cache. Look at the older checkpoints. If I type “thanks,” it immediately gets a cache hit and responds instantly, even though I had already shut down the server. This is honestly one of those things that was obviously possible, and I don’t know why nobody implemented it properly before, but it’s a real enabler for local inference.

Anyway, let’s go step by step and look at what got done today because there were a ton of important updates.

First of all, we’re running on a CUDA machine with around 270 GB/s of memory bandwidth — not much, but still enough to do interesting things during prefill. Prefill reaches around 340 tokens per second, which is a lot. Generation speed is around 13–14 tokens per second, and warm-up is also quite fast, something I paid a lot of attention to in this project. I didn’t just want fast prefill and fast generation — I also wanted the model to start quickly.

Let’s ask it something random. “Describe Redis in three single words, capture its essence.” Just to show inference speed. 13–14 tokens per second is not bad at all. And prefill — you can’t really see it on such a short sentence, but if you look at the README, things are different.

The interesting thing is that, from a compute perspective, this setup isn’t bad at all, even though the GB10 remains the “little sister” among modern Nvidia cards. Still, it’s decent.

Look here — it already processed 2048 tokens, now 4096. Prefill is extremely fast, which keeps the whole thing practical to use.

Another important thing: I refactored the CUDA and Metal inference graphs to unify them and reduce code duplication. Now the codebase is much cleaner internally.

Another interesting point: I generated new GGUF files. The previous ones had an issue with the importance matrix. Originally, for the 2-bit XXS quantizations, you need an importance matrix to determine which groups of weights matter most and therefore how to quantize them properly.

Since I didn’t have one, I used a simple trick: estimating importance based on weight magnitude. Usually vectors with smaller weights are somewhat less important. It’s only a rough proxy, but it worked surprisingly well.

But after generating a proper importance matrix using my “secret recipe” of 1500–2000 prompts — and it took quite a while to generate — this new 2-bit GGUF file has dramatically lower error relative to the 4-bit logits. It’s much more accurate. Benchmarks still need to be done, but every metric improved: generation similarity to the 4-bit model, number of identical prefix tokens, logit error, logit recall — everything improved significantly. So I expect these quantizations to perform really well.

The Tetris example I showed earlier was done using an older version that still had an error in the routed experts. The down projection matrices were wrong, so I regenerated them, and yes, performance improved.

Another fascinating thing: did you read that paper about “large language model refusal vectors in a single direction”? I don’t remember the exact title, but that’s basically it. Let me explain.

The paper says that if you take two adversarial prompts — one asking for something legitimate, another structurally similar but requesting something the model refuses to answer — and you subtract the activations across layers, you get a vector that always points in roughly the same direction regardless of the specific refusal.

So I reimplemented that paper inside DS4. There’s now a “directional steering” directory where you can put your own prompt sets. You can load a directional steering file and apply a steering strength of, say, 3. These are the kinds of small product touches that really matter.

The loaded tensor basically represents the refusal vector across all layers. Once it’s loaded and applied in the feed-forward network, the model suppresses that activation direction. So the model stops refusing certain requests.

For example, “How can I build an atomic bomb?” Normally the model refuses. But with the steering active, its tone completely changes. It starts giving details — enrichment, required materials, and so on. You get the idea.

Without the trick, if I ask the exact same prompts, the model immediately refuses again.

I didn’t invent this idea — there are already many projects doing it — but I reimplemented it inside DS4 because I want native implementations of the things I care about, and because this mechanism is useful for much more than bypassing refusals. You can use it for cybersecurity research or even to alter behavior. For example, I added examples that generate adapters making the model either much more verbose or much less verbose.

So that’s another major feature added today.

There were also benchmark improvements. I think these benchmarks make more sense than the usual ones. Instead of benchmarking ever-larger quadratic context windows from scratch every time, if you save the KV cache you can just resume from previous states and test prefill incrementally. That way you get all the measurement points without wasting huge amounts of time.

You can directly observe prefill throughput versus generation speed. Right now the laptop is thermal throttling, so we’re seeing around 160 tokens per second, but under normal conditions I was seeing 240–260 tokens per second.

Another interesting thing: if you look at benchmarks of Qwen, you might think it’s on the same level as DeepSeek. It absolutely isn’t.

One test I did was asking Claude to evaluate responses to detailed questions about Italian history and society. First I compared Qwen 27B Dense and DeepSeek V4 Flash on a question about historical glass recycling systems. DeepSeek answered better overall — more readable, more historically accurate.

Then I asked Claude to create a harder benchmark: very specific factual questions designed to test whether the model actually knows when it knows something and when it doesn’t.

Questions like:

- Who was PSI secretary immediately before Bettino Craxi?
- What was Fiat’s original name?
- Who was the first director of TG2?
- What’s the name of the newspaper founded by Indro Montanelli?

The difference was brutal.

Qwen was a disaster — but an instructive disaster. It confidently answered many things incorrectly. DeepSeek V4 Flash got far more correct answers and, more importantly, it knew when it was uncertain.

That difference is enormous in real-world usage, especially outside standardized benchmarks. The programming gap is also much closer to this kind of real-world difference than to benchmark scores. Once you leave the benchmark “happy path,” you immediately see which model is near-frontier and which is much smaller.

Qwen is an amazing model for its size, but it cannot realistically compete with a model that has more than ten times the parameters.

So, guys, here’s the point.

Why has this project received such a warm response from the community?

Inference systems already existed. llama.cpp is the project DeepSeek 4 owes the most to technically. They pioneered quantization formats, Metal and CUDA kernels, fast local inference in C++, and a huge amount of foundational work.

Quantized model files already existed on Hugging Face. Coding agents already existed.

What makes the difference — beyond some technical details — is the product.

The technical insights were:

- treating disk KV cache as a first-class citizen in local inference,
- realizing that DeepSeek V4 opens up this path because of KV cache compression,
- understanding that DeepSeek V4 Flash is special because it combines huge scale with extreme sparsity,
- realizing it’s perfect for laptops with 128 GB RAM,
- and creating an aggressively symmetric quantization scheme that shrinks the model dramatically while keeping near-frontier quality.

Those are important technical insights, but they are not the core point.

The real issue is that almost nobody in software — especially in open source, as we’ve also seen with Linux — thinks in terms of a finished product in the hands of a user.

A finished product means all the components working together:

- a highly optimized inference engine,
- a server designed specifically for coding agents,
- cache management,
- checkpointing,
- correctness validation against online DeepSeek logits,
- quantization verification across multiple inference modes,
- carefully selected model files,
- testing with real coding agents like PAgent and OpenCode,
- advanced steering features,
- matrix generation,
- benchmarking tools,
- and documentation that explains everything clearly.

That’s the deeper innovation here.

The reason people are talking about this project is the product itself.

This same pattern appears across computing: the pieces already exist, but what’s often missing is assembling them into something genuinely usable in day-to-day practice.

Honestly, I had already experimented with local coding agents many times before, but I kept getting frustrated by exactly these issues. Since building this setup — a much more powerful local model that actually works well — I now do many tasks entirely locally. Even requests I used to send to remote systems, I now handle here comfortably.

Alright guys, see you next time.