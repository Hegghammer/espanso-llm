# llm Espanso Package

This package extends [Espanso](https://espanso.org) to allow querying and receiving output from large language models wherever you are writing. It builds on @rohitna's [chatgpt-script](https://github.com/rohitna/chatgpt-script), adding support for web search, link following, and multiple LLM configurations.

The package works by sending whatever is in your clipboard along with a preset prompt to the LLM of your choice. You can configure up to three different LLMs, from any provider, including local ones. The package also lets you add web search and link following to any model, meaning you can ask things like "Which NHL games are on this weekend" or "Summarize the content of this webpage: https://example.com" even from models that don't have built-in web access. 

A simple example would go something like this:

1. Write "Why is the sky blue?" in any text editor.
2. Select the question and type CTRL+C to copy it to the clipboard.
3. On the line below, type your chosen trigger phrase, for example `,ask`. Espanso will insert the response from the LLM into your text editor where your cursor is. 

The package comes with a syntax that lets you associate any trigger phrase with any prompt, so you could have commands like `,debug`, `,summarize` or `,translate`, each of which would do something different with the text in your clipboard. The folder `/prompts` contains a number of examples that you can tweak to your needs. 

## Installation

The package currently only works on Linux and MacOS. I hope to add Windows compatibility in due course.

1. Install Espanso.

- [Linux](https://espanso.org/docs/install/linux/)
- [MacOS](https://espanso.org/docs/install/mac/)

2. Install [Python](https://realpython.com/installing-python/).

3. Install the `llm` package.

```
espanso install llm --git https://gitlab.com/hegghammer/espanso-llm --external
```

4. Install the [`llm-cli` program](https://github.com/Hegghammer/llm-cli)

```bash
git clone https://github.com/Hegghammer/llm-cli.git
cd llm-cli
pip install .
```

5. Add the path of `llm-cli` to `config.ini`.

```bash
sed -i "s|/path/to/llm-cli/executable|$(which llm-cli)|" $(espanso path packages)/llm/config.ini
```

6. Add the correct paths to the database and log files in `config.ini`.

```bash
sed -i "s|/path/to/dbfile|$(espanso path packages)/llm/logs/chats.db|" $(espanso path packages)/llm/config.ini
sed -i "s|/path/to/logfile|$(espanso path packages)/llm/logs/log.txt|" $(espanso path packages)/llm/config.ini
```

7. Configure the LLMs.

Open `config.ini` in a text editor. You will see that there are three "slots" for LLMs, each with a set of parameters:

- `*_name`: The name you want to give to the configuration. 
- `*_endpoint`: The URL of the API endpoint, for example `https://api.mistral.ai/v1/chat/completions`.
- `*_model`: The name of the model, for example `mistral-large-latest`.
- `*_api_key`: Your API key for this provider.
- `*_temperature`: A number from 0 (less creative) to 2. 

8. (Optional) Configure RAG.

With `config.ini` still open in a text editor, fill in one or more of the three slots for RAG. 

- `*_doc_dir`: The directory containing the documents you want to query. NB: plaintext documents only.
- `*_db_dir`: A new directory for the Chroma vector database for the collection 
- `*_excerpt_length`: The length (in characters) of the excerpts that the LLM cites as sources for its answer.
- `*_source_linkformat`: The format of the file links that the LLM provides when citing sources. The options are "markdown" and "wikilinks". The latter is useful if you are quering a [Foam](https://foambubble.github.io/foam/) or [Obsidian](https://obsidian.md) notes collection.
- `*_temperature`: A number from 0 (less creative) to 2.

9.  Restart Espanso.

```bash
espanso restart
```

You should now be good to go. Check that the package is working by opening an editor and typing `,test`. This should print out the help menu of `llm-cli`. You can also test the first LLM by typing a question (e.g. "Which model are you?"), copying the question to the clipboard, and typing `,ask`.

If Espanso gives you an error message, check the logs with `espanso log`.

## Configuration

### Providers

You can use any LLM provider with an API endpoint. See the provider documentation for how to obtain the the endpoint URL, the model name, and the API key. You can also use a local model served by platforms such as [Ollama](https://ollama.com) or [XInference](https://inference.readthedocs.io/en/latest/). 

For web search, the package is set up to use Jina AI Search API by default. The service is free and requires no registration of personal details. You do need an API key to put in `config.ini`, but it is easily obtained by going to https://jina.ai and by clicking first on the "API" button and then on "API key and billing". You can use another search API provider if you like; just replace the endpoint URL and API key. You can even set up a local [Searxng](https://docs.searxng.org) instance and use that as the backend. Note that the package does now allow for supplying additional search parameters like language and location, because all the search API providers use different protocols and I wanted to prioritize provider neutrality. For more control, you must tweak the parameters of the `web_search()` function in `/scripts/main.py`.

For link following (page reading), the package uses Jina AI Reader API by default. If you use Jina for search, you can use the same API key for link following. Other services may work, but I have not tested them.

RAG is currently designed primarily for models served locally with Ollama. Other options may be added in the future.

### Built-in triggers

The package comes with a number of sample triggers that should work out of the box. The most important ones are:

- `,ask`: Send the clipboard contents to your default LLM (aka LLM1, the one under `provider_1` in `config.ini`)
- `,webask`: Ask LLM 1, but with web search first.
- `,linkask`: Ask LLM 1, but with link following. For example, the prompt "Summarize this article: https://example.com/article" followed by `,fask` will look up the web page and summarize it.
- `,sum`: Ask LLM1 to summarize the clipboard contents. 
- `,debug`: Ask LLM1 to debug the code in the clipboard contents. 
- `,tra-fr`: Ask LLM1 to translate the clipboard contents into French

To ask the LLMs of `provider_2` and `provider_3` in `config.ini`, prepend the number to the above triggers, for example:
- `,2ask`
- `,3ask`
- `,2webask`
- `,3debug`
- and so forth.

To see all the preconfigured triggers, browse the `.yml` files in the `/prompts` folder.

### Trigger customization

You may well wish to tailor the triggers to suit your needs. If you have perused the example `yml` files in `/prompts`, you will have seen that they mostly follow this template:

```yaml
matches:
  - trigger: TRIGGER_PHRASE
    replace: "{{output}}"
    vars:
      - name: output
        type: shell
        params:
          cmd: COMMAND
```

To understand what is going on, it helps to know that the heart of the package is the `llm-cli` program, which accepts a variety of arguments and executes a call to an LLM endpoint based on the arguments it receives. So the `COMMAND` variable after `cmd:` in the last line above is essentially a shell command that runs `llm-cli` with your chosen parameters, like so:

```bash
/path/to/llm-cli --parameter value
```

But this construction can get long and complicated when all the parameters are spelled out. The package therefore defines -- in `package.yml` -- a variety of global variables that serve as building blocks for `COMMAND`. For example, it defines the variable `{{P1_CALL}}` as a shorthand for the script

```bash
/path/to/llm-cli \
--provider-endpoint endpoint1
--model model1
--api_key api-key1
```

This then lets you simply write `{{P1_CALL}}` when you want to invoke the LLM in the `provider_1` slot. Hence the COMMAND of `,ask` can be as short as this:

```yaml
cmd: |
  {{P1_CALL}} \
  --temperature {{P1_TEMP}} \
  --system-role "A helpful assistant" \
  --prompt "Answer this concisely"
```

The `--prompt` parameter is important; this is where you can really steer the LLM to do what you want. 

To see all the available parameters, run `llm-cli -h` in your terminal or simply type `,test` in any text editor.

### Cost control

If you are calling an expensive LLM, be careful how you use the package. Note in particular that your conversations are stored in memory for as many minutes as you set in the parameter `conversation_timeout_minutes` in `config.ini` (default 15 minutes). Moreover, each web search returns up to 10,000 tokens (even more with the `--full-content` parameter), all of which are passed to the LLM. Tokens can easily accumulate this way, so you may want to occasionally clear the memory with the default trigger `,cdb` (for "clear database").
