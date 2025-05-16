# Chapter 5: LLM Interaction (call_llm)


```markdown
# Chapter 5: LLM Interaction (call_llm)

In the previous chapter, [Chapter 4: BatchNode](04_batchnode_.md), we learned how to use `BatchNode` to efficiently process multiple items, like generating multiple chapters for our tutorial. But how do we actually *generate* the content for each chapter? That's where interacting with a Large Language Model (LLM) comes in! In this chapter, we'll dive into the `call_llm` function, the key to making this happen.

## What is `call_llm`?

`call_llm` is our way of talking to a "smart assistant" (the LLM). It's a function that takes our instructions and the context (like code snippets or the chapter structure), sends them to the LLM, and gets back the generated text for our tutorial. Think of it as a phone call: we tell the assistant what we need, and it gives us the answer.

## Key Concepts

*   **Prompt:** The instructions we give the LLM. This is like the question or task we want the LLM to perform.
*   **LLM (Large Language Model):** The "smart assistant" that understands our prompt and generates text. We're using a powerful AI model to help us write the tutorial.
*   **Response:** The text the LLM generates in response to our prompt. This is the content for our chapter!
*   **Cache (Optional):** To avoid repeatedly calling the LLM with the same prompt (which can be slow and expensive), we use a cache.  The cache stores the LLM's responses so that if we ask the same question again, we can get the answer instantly.

## Using `call_llm` to Generate a Tutorial Chapter

Let's consider the `WriteChapters` BatchNode that we saw in the last chapter.  Within `WriteChapters`, the `call_llm` function is the magic that turns our instructions into the actual Markdown content for each chapter:

```python
# Inside the WriteChapters.exec() method (simplified)
chapter_content = call_llm(prompt)
```

That's it! We give `call_llm` a `prompt` (which we'll see how to build shortly), and it returns the content for the chapter.

Let's examine a simplified view of a prompt:

```
Write a very beginner-friendly tutorial chapter...

Concept Details:
- Name: Flow
- Description: ...

Complete Tutorial Structure:
1. [Flow](01_flow_.md)
2. [File Crawling](02_file_crawling_.md)
...

Relevant Code Snippets:
```python
from pocketflow import Flow
...
```

Instructions:
... (e.g., explain the concept, give code examples) ...
```

In this prompt, we are providing all the information needed to generate the content, and asking `call_llm` to write content with the right structure!

## Under the Hood: How `call_llm` Works

Here's a simplified look at the `call_llm` function's implementation:

```python
# utils/call_llm.py (simplified)
import os
import json
from google import genai  # Or another LLM library

cache_file = "llm_cache.json"

def call_llm(prompt: str, use_cache: bool = True) -> str:
    # 1. Check the Cache (if enabled)
    if use_cache:
        # Try to load the cache from disk
        # and look up the prompt in the cache

    # 2.  Call the LLM (if not in cache)
    #   - Get the API Key and set up the model
    #   - Send the prompt to the LLM
    #   - Get the response from the LLM

    # 3. Save the Response to the Cache (if enabled)
    if use_cache:
        # Save the response to the cache on disk

    # 4. Return the Response
    return response_text
```

Here's a simplified explanation, let's pretend we're the `call_llm` function:

1.  **Check the Cache:** "Hey, have I answered this question before?" If yes, great! I can give that same answer without making another call to the LLM.
2.  **Call the LLM:** If the answer isn't in the cache, I need to talk to the LLM.
    *   I send the `prompt` (our instructions and context) to the LLM.
    *   The LLM analyzes the prompt and returns the text for our chapter.
3.  **Save to Cache:** If the cache is enabled, I save the response (the chapter content) to the cache so I can reuse it later.
4.  **Return the Response:** I return the chapter content.

Here is a sequence diagram to better explain this

```mermaid
sequenceDiagram
    participant User
    participant call_llm
    participant Cache (Optional)
    participant LLM
    User->>call_llm: Call with prompt
    activate call_llm
    alt use_cache == True
        call_llm->>Cache: Check for prompt
        activate Cache
        alt Prompt in Cache
            Cache-->>call_llm: Response from cache
            deactivate Cache
            call_llm-->>User: Response from cache
        else Prompt not in Cache
            Cache-->>call_llm: Prompt not found
            deactivate Cache
            call_llm->>LLM: Send prompt
            activate LLM
            LLM-->>call_llm: Response
            deactivate LLM
            call_llm->>Cache: Save response
            activate Cache
            Cache-->>call_llm: Saved
            deactivate Cache
            call_llm-->>User: Response from LLM
        end
    else use_cache == False
        call_llm->>LLM: Send prompt
        activate LLM
        LLM-->>call_llm: Response
        deactivate LLM
        call_llm-->>User: Response from LLM
    end
    deactivate call_llm
```

## Code Example and Explanation

Let's look at some key parts of the code, focusing on the `call_llm` implementation using the Google Gemini API. Note that you'll need an API key to run this.

```python
# utils/call_llm.py (excerpt)
from google import genai  # Importing the library
import os
import json
import logging
from datetime import datetime

# ... (Logging and Cache Setup – see earlier in this chapter) ...

# By default, we Google Gemini 2.5 pro, as it shows great performance for code understanding
def call_llm(prompt: str, use_cache: bool = True) -> str:
    # Log the prompt
    logger.info(f"PROMPT: {prompt}")

    # Check cache if enabled
    if use_cache:
        # Load cache from disk
        cache = {}
        if os.path.exists(cache_file):
            try:
                with open(cache_file, "r") as f:
                    cache = json.load(f)
            except:
                logger.warning(f"Failed to load cache, starting with empty cache")

        # Return from cache if exists
        if prompt in cache:
            logger.info(f"RESPONSE: {cache[prompt]}")
            return cache[prompt]

    # # Call the LLM if not in cache or cache disabled
    # client = genai.Client(
    #     vertexai=True,
    #     # TODO: change to your own project id and location
    #     project=os.getenv("GEMINI_PROJECT_ID", "your-project-id"),
    #     location=os.getenv("GEMINI_LOCATION", "us-central1")
    # )

    # You can comment the previous line and use the AI Studio key instead:
    client = genai.Client(
        api_key=os.getenv("GEMINI_API_KEY", ""),
    )
    model = os.getenv("GEMINI_MODEL", "gemini-2.5-pro-exp-03-25")
    # model = os.getenv("GEMINI_MODEL", "gemini-2.5-flash-preview-04-17")
    
    response = client.models.generate_content(model=model, contents=[prompt])
    response_text = response.text

    # Log the response
    logger.info(f"RESPONSE: {response_text}")

    # Update cache if enabled
    if use_cache:
        # Load cache again to avoid overwrites
        cache = {}
        if os.path.exists(cache_file):
            try:
                with open(cache_file, "r") as f:
                    cache = json.load(f)
            except:
                pass

        # Add to cache and save
        cache[prompt] = response_text
        try:
            with open(cache_file, "w") as f:
                json.dump(cache, f)
        except Exception as e:
            logger.error(f"Failed to save cache: {e}")

    return response_text
```

Let's break this down:

1.  **Import Libraries:** We import `google.genai` (you'll need to install the `google-generativeai` package). This is how we communicate with the LLM.
2.  **Check the Cache:**
    *   The code checks if we've already asked the same question (prompt) before. If we have, it returns the cached answer.
3.  **Call the LLM:**
    *   It sets up the Gemini model using your API key (`os.getenv("GEMINI_API_KEY", "")`). The Gemini API Key should be specified as an Environment Variable.
    *   It then calls the LLM using `client.models.generate_content()`, passing in the `prompt`.
    *   It gets the LLM's `response_text`.
4.  **Save to Cache (if enabled):** If the cache is enabled, the code saves the `prompt` and `response_text` to a cache file (`llm_cache.json`) so it can be reused later.
5.  **Return the Response:**  Finally, it returns the generated `response_text` (the chapter content).

To use this in your code, you can just call `call_llm(your_prompt)` where `your_prompt` is the string containing your instructions, context, and requests.

## Conclusion

The `call_llm` function is the core of our tutorial generation system. It allows us to communicate with the Large Language Model (LLM) and get the content for our chapters.  We've learned about the `prompt`, the LLM, the response, and how the optional cache helps make the process faster and more efficient.

Now that we understand how to interact with the LLM to generate chapter content, in the next chapter, we will review and summarize all we learned.
```

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)