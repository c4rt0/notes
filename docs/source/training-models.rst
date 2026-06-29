Training your own models
========================

A from-zero guide to fine-tuning an open model on a domain you care about
(the running example: Fedora CoreOS knowledge).

Why fine-tune?
--------------

General models know a bit of everything but not your niche. You can paste docs
into every prompt, but that's tedious and burns tokens. **Fine-tuning** bakes
the knowledge in.

.. list-table::
   :header-rows: 1

   * - Situation
     - Approach
   * - One-off question
     - paste the doc into the prompt
   * - Recurring questions in one domain
     - fine-tune
   * - Want a specific output format every time
     - fine-tune
   * - Not sure what you want yet
     - prompt-engineer first

**Rule of thumb:** if you copy-paste the same context into every conversation,
that context should be trained into the model.

Core concepts
-------------

- **Model** - a math function with billions of adjustable **parameters/weights**
  that predicts the next token.
- **Pre-training** - done once by big labs on the whole internet (millions of $).
  You don't do this.
- **Fine-tuning** - take an already-trained model and adjust it on *your* data
  ($1-50, hours).
- **LoRA** (Low-Rank Adaptation) - freeze the original model and train tiny
  adapter layers (~0.1-1% of the size). Less memory, faster, preserves existing
  knowledge, and the adapter is tiny (~50-200 MB) and shareable.
- **QLoRA** - LoRA on top of a model loaded in 4-bit precision; lets you
  fine-tune a 7-8B model on a single consumer / free-Colab GPU.
- **Dataset** - instruction/response pairs you train on. Quality > quantity:
  500 excellent examples beat 50,000 sloppy ones (the model learns your
  mistakes too).
- **Evaluation** - hold out 10-20% of data, plus a fixed set of questions you
  know the answers to, and compare before/after.

Choosing a base model
---------------------

.. list-table::
   :header-rows: 1

   * - Model
     - Size
     - Min GPU (QLoRA)
     - Good for
   * - Llama 3.1 8B Instruct
     - 8B
     - 8 GB VRAM
     - best quality for the size - start here
   * - Mistral 7B
     - 7B
     - 8 GB VRAM
     - fast, good at code
   * - Phi-3 Mini
     - 3.8B
     - 4 GB VRAM
     - tiny but capable
   * - Llama 3.1 70B
     - 70B
     - 40 GB+ VRAM
     - much better, serious hardware

Gathering training data
-----------------------

The most important and time-consuming step. Convert raw sources (docs, GitHub
issues, discussions, mailing lists, example configs, tests) into
instruction/response pairs:

.. code-block:: json

        {
          "instruction": "How do auto-updates work in Fedora CoreOS?",
          "response": "FCOS uses Zincati (the update agent), Cincinnati (the update server that serves the update graph) and OSTree (updates delivered as commits, staged then rebooted into). Configure strategy in /etc/zincati/config.d/."
        }

A starting point to export GitHub issues, then **manually review and clean** -
drop "me too" answers, wrong/outdated answers, and over-specific setups:

.. code-block:: python

        import json, subprocess

        def export_issues(repo, label="question", limit=200):
            nums = subprocess.run(
                ["gh", "api", f"repos/{repo}/issues", "--paginate",
                 "-q", f'.[] | select(.labels[].name=="{label}") | '
                       f'select(.comments>0) | .number'],
                capture_output=True, text=True).stdout.split()[:limit]
            pairs = []
            for n in nums:
                issue = json.loads(subprocess.run(
                    ["gh","api",f"repos/{repo}/issues/{n}"],
                    capture_output=True, text=True).stdout)
                comments = json.loads(subprocess.run(
                    ["gh","api",f"repos/{repo}/issues/{n}/comments"],
                    capture_output=True, text=True).stdout)
                if comments:
                    pairs.append({"instruction": f"{issue['title']}\n{issue['body'][:500]}",
                                  "response": comments[0]["body"]})
            json.dump(pairs, open(f"data_{repo.replace('/','_')}.json","w"), indent=2)

How much data? ~50-100 examples gives a noticeable bump on narrow tasks;
200-500 gives good focused domain knowledge; 1000+ a strong specialist with
diminishing returns. Start with ~200 high-quality pairs.

Fine-tuning with QLoRA
----------------------

.. code-block:: bash

        pip install transformers datasets peft bitsandbytes accelerate trl

.. code-block:: python

        import torch
        from transformers import (AutoModelForCausalLM, AutoTokenizer,
                                   BitsAndBytesConfig, TrainingArguments)
        from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training
        from trl import SFTTrainer

        # 1. load base model in 4-bit (QLoRA)
        bnb = BitsAndBytesConfig(load_in_4bit=True, bnb_4bit_quant_type="nf4",
                                 bnb_4bit_compute_dtype=torch.bfloat16,
                                 bnb_4bit_use_double_quant=True)
        name = "meta-llama/Meta-Llama-3.1-8B-Instruct"
        tok = AutoTokenizer.from_pretrained(name); tok.pad_token = tok.eos_token
        model = AutoModelForCausalLM.from_pretrained(name, quantization_config=bnb,
                                                     device_map="auto")
        model = prepare_model_for_kbit_training(model)

        # 2. LoRA adapters
        lora = LoraConfig(r=16, lora_alpha=32, lora_dropout=0.05, bias="none",
                          task_type="CAUSAL_LM",
                          target_modules=["q_proj","k_proj","v_proj","o_proj",
                                          "gate_proj","up_proj","down_proj"])
        model = get_peft_model(model, lora)
        model.print_trainable_parameters()   # ~0.17% trainable

        # 3. + 4. train
        args = TrainingArguments(output_dir="./out", num_train_epochs=3,
                                 per_device_train_batch_size=4,
                                 gradient_accumulation_steps=4, learning_rate=2e-4,
                                 fp16=True, optim="paged_adamw_8bit", report_to="none")
        SFTTrainer(model=model, args=args, train_dataset=ds["train"],
                   eval_dataset=ds["test"], tokenizer=tok,
                   dataset_text_field="text", max_seq_length=2048).train()
        # 5. save just the adapter (~50-200 MB)

Format each example to the model's chat template and split
``dataset.train_test_split(test_size=0.1)`` before training. To use the result,
load the base model and apply the adapter with ``PeftModel.from_pretrained``.

Hardware and cost
-----------------

For QLoRA on an 8B model with ~200 examples: an 8 GB+ VRAM GPU, 16 GB RAM,
~20 GB disk, 30-90 minutes for 3 epochs - $0 on Colab's free T4, or ~$2-5 on a
cloud GPU (Lambda, RunPod). Check your GPU with ``nvidia-smi``.

Fine-tuning vs RAG
------------------

RAG (Retrieval-Augmented Generation) keeps docs in a vector DB and retrieves
chunks at query time. Prefer **RAG** when the knowledge changes often, you need
citations, or you must use a proprietary model. Prefer **fine-tuning** when you
want knowledge baked in (no retrieval latency), a specific style/format, or
local/offline use. You can combine both: fine-tune for style + base knowledge,
RAG for up-to-date specifics.

Things to remember: fine-tuning shifts default behavior, it doesn't add
abilities the base model lacks; data quality is everything; start small (50
examples, evaluate, add more) rather than collecting thousands before training
once.

Glossary
--------

.. list-table::
   :header-rows: 1

   * - Term
     - Meaning
   * - Parameters / weights
     - the adjustable numbers inside the model
   * - LoRA / QLoRA
     - train tiny adapters / + load the base in 4-bit
   * - Epoch
     - one complete pass over the training data
   * - Learning rate
     - how aggressively weights adjust (``2e-4`` is a good start)
   * - Loss
     - how wrong predictions are; training drives it down
   * - Overfitting
     - memorizing training data (low train loss, high eval loss)
   * - VRAM
     - GPU memory - usually the training bottleneck
   * - Quantization
     - storing weights in lower precision (4-bit) to save memory
   * - Inference
     - using the trained model (vs training it)
   * - RAG
     - retrieving relevant docs at query time instead of training them in
   * - Hugging Face / PEFT
     - the model hub + ``transformers`` / the library implementing LoRA
