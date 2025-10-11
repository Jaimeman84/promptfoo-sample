# File-Based Promptfoo Example

This example demonstrates a modular approach to organizing promptfoo configurations using external files.

## Directory Structure

```
modular-vadim/
├── promptfooconfig.yaml     # Main configuration file
├── prompts/                 # Prompt templates
│   ├── customer_service.txt
│   ├── technical_support.txt
│   └── providers.yaml       # Model providers
└── tests/                   # Test cases
    └── common/
        └── var.yaml         # Variables and assertions
```

## Setup

1. Install promptfoo globally:

```bash
npm install -g promptfoo
```

2. Set up your environment variables:

```bash
# Copy the example env file
cp env.example .env

# Edit the .env file with your API keys
```

## Running Evaluations

Run the evaluation from the modular-vadim directory:

```bash
cd modular-vadim
promptfoo eval
```

View the results:

```bash
promptfoo view
```

## Configuration Details

- **Prompts**: Text files with templates for customer service and technical support responses
- **Providers**: YAML file listing the LLM providers to test against
- **Tests**: YAML file with test variables and assertions for different customer issues

This modular structure makes it easy to:
- Add new prompt templates without modifying the main config
- Switch between different sets of providers
- Organize test cases by category