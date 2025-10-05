# promptfoo-sample

A sample project for evaluating and testing prompts using promptfoo.

## Cloning the Repository

```bash
git clone https://github.com/jaimeman84/promptfoo-sample.git
cd promptfoo-sample
```

## Installation

1. Install promptfoo:

```bash
npm install -g promptfoo
```

2. Set up your API keys:

You can set environment variables directly:

```bash
# For OpenAI models
export OPENAI_API_KEY=your_openai_api_key
# Add other provider keys as needed
```

Or create a `.env` file in the project root:

```
# .env file
OPENAI_API_KEY=your_openai_api_key
# Add other provider keys as needed
```

## Getting Started

1. Edit `promptfooconfig.yaml` to customize your prompts and tests.

2. Run the evaluation:

```bash
promptfoo eval
```

3. View the results:

```bash
promptfoo view
```

## Configuration

The project includes a sample `promptfooconfig.yaml` file that demonstrates:
- Multiple prompt templates
- Different model providers
- Various assertion types for testing outputs

Refer to the [promptfoo documentation](https://promptfoo.dev/docs/) for more details on configuration options.