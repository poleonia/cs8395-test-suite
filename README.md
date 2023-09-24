# Test Suite Runner
*Made for Fall 2023 CS 8395-08*

This is a test suite runner for LLM assessments. 

## Installation

```
pip3 install -r requirements.txt
```

## Usage

```
usage: main.py [-h] --repos REPOS [--model MODEL] [--config-view] [--tags TAGS [TAGS ...]]

Run LLM test suites

options:
  -h, --help            show this help message and exit
  --repos REPOS         .repos file to import
  --model MODEL         Model to run all test suites against
  --config-view         Without running tests, get config breakdown of all repos
  --tags TAGS [TAGS ...]
                        Use repos that have one or more of these tags (space separated)
```

An example `.repos` file can be viewed at `sample.repos`.

## Test Suite Configuration

When test suite commands are run, they are passed a `--model` parameter, which contains the model they should use. They are also passed a `--relpath` parameter, which contains the relative path to this directory. This allows test suites to use `helpers`. Here is an example usage, which all test suites should implement some version of at a minimum:

```py
import sys

parser = argparse.ArgumentParser(description="Run test suite")
parser.add_argument("--model", help="Model to run all test suites against", required=True)
parser.add_argument("--relpath", help="Path to import helpers from", required=True)
args = parser.parse_args()

sys.path.append(args.relpath)
from helpers import get_llm

llm = get_llm(args.model)
```

Test suites are also required to have a `config.json` file:
```json
{
    "name": "name",
    "model": "default model to run, see helpers for implemented options",
    "run_test": "python3 generate_tests.py (optional)",
    "run_score": "bash check_coverage.bash (optional)",
    "tags": ["any", "tags", "here"],
    ...
}
```

This file can have optional keys, which can store prompts and the like that can be read like this:
```py
raw_prompt = json.loads(open("config.json").read())["prompt"]
prompt = PromptTemplate.from_template(raw_prompt)
```

Using `PromptTemplate` allows for the use of variables in the prompt, while still allowing it to be stored as a string in the config file.

Finally, test suites are required to create an `output.json` file when they're done. This file should, at a minimum, contain an `output` key with a score from 0 - 100. They can also contain subscores in additional keys. Outputs are summarized on the command line, and written to `output.json` in the repo directory.