# llm Espanso Package

This package extends [Espanso](https://espanso.org) to allow querying and receiving output from large language models wherever you are writing. It builds on @rohitna's [chatgpt-script](https://github.com/rohitna/chatgpt-script), adding support for web search, link following, multiple LLM configurations, and retrieval-augmented generation.

The package works by sending whatever is in your clipboard along with a preset prompt to the LLM of your choice. You can configure up to three different LLMs, from any provider, including local ones. The package also lets you add web search and link following to any model, meaning you can ask things like "Which NHL games are on this weekend" or "Summarize the content of this webpage: https://example.com" even from models that don't have built-in web access. 

A simple example would go something like this:

1. Write "Why is the sky blue?" in any text editor.
2. Select the question and type CTRL+C to copy it to the clipboard.
3. On the line below, type your chosen trigger phrase, for example `,ask`. Espanso will insert the response from the LLM into your text editor where your cursor is. 

The package comes with a syntax that lets you associate any trigger phrase with any prompt, so you could have commands like `,debug`, `,summarize` or `,translate`, each of which would do something different with the text in your clipboard. The folder `/prompts` contains a number of examples that you can tweak to your needs. 

## Installation

Espanso-llm works on all three major operating systems, but the procedure for Windows is slightly different in places (I will point those places out.)

**1. Install [Espanso](https://espanso.org/install/).**

**2. Install the `llm` package.**

```
espanso install llm --git https://github.com/hegghammer/espanso-llm --external
```

**3. Install the Python program [`llm-cli`](https://github.com/Hegghammer/llm-cli)**

This assumes you have [Python](https://realpython.com/installing-python/) (>=3.8,<3.12) installed.

```
git clone https://github.com/Hegghammer/llm-cli.git
cd llm-cli
pip install .
```

**4. Add paths to llm-cli executable, `chats.db` and `log.txt` in `config.ini`.**

You must provide Espanso-llm with system-specific paths to the `llm-cli` executable, the database, and the log file.

On Linux and MacOS you can simply run this command:

```sh
sed -i "s|/path/to/llm-cli/executable|$(which llm-cli)|" $(espanso path packages)/llm/config.ini
sed -i "s|/path/to/dbfile|$(espanso path packages)/llm/logs/chats.db|" $(espanso path packages)/llm/config.ini
sed -i "s|/path/to/logfile|$(espanso path packages)/llm/logs/log.txt|" $(espanso path packages)/llm/config.ini
```

On Windows this is best done manually. 

a. In a terminal, run `where llm-cli` to get the path to the llm-cli executable on your system. Copy it to the clipboard

b. Open `config.ini` in Notepad. On line 2, paste in the path to the llm-cli executable to the right of the equal sign, replacing `/path/to/llm-cli/executable`.

c. Back in the terminal, run `espanso path packages` to output the path to the Espanso packages folder. Copy it to the clipboard. 

d. Back in `config.ini` in Notepad, on line 3, replace `/path/to/dbfile` with the path to the Espanso packages folder in your clipboard. Append `/llm/logs/chats.db` to the path.

e. On line 4, replace `/path/to/logfile` with the path to the Espanso packages folder in your clipboard. Append `/llm/logs/log.txt` to the path.

**4a. (Windows only): Replace `package.yml` and `/prompts`** 

Windows users must also replace `package.yml` with the Windows-compatible version supplied with the package. 

a. Delete `package.yml`

b. Rename `package.yml.windows` to `package.yml`

You also need to replace the prompts folder with the Windows-compatible one. 

a. Delete `/prompts`

b. Rename `/prompts.windows` to `/prompts`

**5. Configure the LLMs.**

Open `config.ini` in a text editor. You will see that there are three "slots" for LLMs, each with a set of parameters:

- `*_name`: The name you want to give to the configuration. 
- `*_endpoint`: The URL of the API endpoint, for example `https://api.mistral.ai/v1/chat/completions`.
- `*_model`: The name of the model, for example `mistral-large-latest`.
- `*_api_key`: Your API key for this provider.
- `*_temperature`: A number from 0 (less creative) to 2. 

**6. (Optional) Configure RAG.**

With `config.ini` still open in a text editor, fill in one or more of the three slots for RAG. 

- `*_doc_dir`: The directory containing the documents you want to query. NB: plaintext documents only.
- `*_db_dir`: A new directory for the Chroma vector database for the collection 
- `*_excerpt_length`: The length (in characters) of the excerpts that the LLM cites as sources for its answer.
- `*_source_linkformat`: The format of the file links that the LLM provides when citing sources. The options are "markdown" and "wikilinks". The latter is useful if you are quering a [Foam](https://foambubble.github.io/foam/) or [Obsidian](https://obsidian.md) notes collection.
- `*_temperature`: A number from 0 (less creative) to 2.

**7.  Restart Espanso.**

```
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
#          shell: powershell # Add this line if on Windows
```

To understand what is going on, it helps to know that the heart of the package is the `llm-cli` program, which accepts a variety of arguments and executes a call to an LLM endpoint based on the arguments it receives. So the `COMMAND` variable after `cmd:` in the last line above is essentially a shell command that runs `llm-cli` with your chosen parameters, like so:

```sh
/path/to/llm-cli --parameter value
```

But this construction can get long and complicated when all the parameters are spelled out. The package therefore defines -- in `package.yml` -- a variety of global variables that serve as building blocks for `COMMAND`. For example, it defines the variable `{{P1_CALL}}` as a shorthand for the script

```sh
/path/to/llm-cli \
--config-file /path/to/config.ini \
--provider-endpoint endpoint1 \
--provider_api_key api-key1 \
--model model1
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

Note that on Windows, you cannot use backslashes for multi-line scripting; you must put the command on a single line. 

To see all the available parameters, run `llm-cli -h` in your terminal or simply type `,test` in any text editor.

### Cost control

If you are calling an expensive LLM, be careful how you use the package. Note in particular that your conversations are stored in memory for as many minutes as you set in the parameter `conversation_timeout_minutes` in `config.ini` (default 15 minutes). Moreover, each web search returns up to 10,000 tokens (even more with the `--full-content` parameter), all of which are passed to the LLM. Tokens can easily accumulate this way, so you may want to occasionally clear the memory with the default trigger `,cdb` (for "clear database").
