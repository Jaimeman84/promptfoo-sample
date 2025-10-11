# Promptfoo Modular Configuration Example

This example demonstrates the modular configuration approach for promptfoo.

## Directory Structure

```
promptfoo-directory-structure/
├── configs/                    # Configuration files
│   ├── prompts.yaml           # Defines all prompt templates
│   ├── providers.yaml         # Defines AI models and settings
│   └── default-test.yaml      # Default assertions for all tests
├── prompts/                   # Prompt template files
│   ├── system-message.txt     # System message prompt
│   └── user-prompt.txt        # User prompt template
├── tests/                     # Test cases organized by category
│   └── accuracy/
│       └── math-problems.yaml # Math test cases
├── promptfooconfig.yaml       # Main config (orchestrator)
├── .gitignore
└── env.example
```

## How It Works

### Main Config (promptfooconfig.yaml)
The main config is the **orchestrator** that:
- References WHERE to find prompts (`file://configs/prompts.yaml`)
- References WHERE to find providers (`file://configs/providers.yaml`)
- References WHERE to find tests (`file://tests/**/*.yaml`)
- References default test settings (`file://configs/default-test.yaml`)

### Purpose of Multiple YAMLs
- **Separation of concerns**: Each file handles one aspect (prompts, providers, tests)
- **Reusability**: Swap different provider configs or test suites easily
- **Organization**: Keep related configurations together
- **Maintainability**: Easier to update specific parts without touching everything

## Setup

1. Install promptfoo:
```bash
npm install -g promptfoo
```

2. Set up environment:
```bash
cp env.example .env
# Edit .env with your API key
```

## Running

```bash
cd promptfoo-directory-structure
promptfoo eval
promptfoo view
```

## Key Concepts

- **Main config**: Orchestrates and references other files
- **Config files**: Define prompts, providers, and default test settings
- **Prompt files**: Store actual prompt templates (can be .txt or inline in YAML)
- **Test files**: Organized by category (accuracy, safety, performance, etc.)
- **Default tests**: Apply to all test cases unless overridden
