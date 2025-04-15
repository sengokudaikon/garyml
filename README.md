# GaryML Language Design Document - v0.1

## 1. Introduction & Motivation

GaryML (Gary Markup Language) is a domain-specific language designed for instructing and interacting with AI agents, particularly in software development and analysis workflows. It aims to bridge the gap between human-readable instructions and the structured input required for reliable AI performance. Inspired by interactive command patterns (like .dotCommands in Aider), configuration file simplicity (like TOML/YAML), and research findings on structured prompting (like XML's effectiveness for LLMs), GaryML provides a consistent syntax for defining context, instructions, plans, executable commands, and specialized data blocks like diffs.

## 2. Goals

- **Human Readability & Writability**: Syntax should be intuitive and easy for developers to read and write manually
- **Machine Parseability**: Structure should be unambiguous and easily parsed by software tools (like agent controllers or IDE extensions)
- **Structured Prompting**: Provide clear sections for different types of information (context, instructions, plan, execution) to improve LLM comprehension and task adherence
- **Command Integration**: Natively represent executable agent commands within the structure
- **Extensibility**: Allow for the definition and use of custom commands and potentially new directives in the future
- **Interactivity Support**: Facilitate workflows where GaryML files define project context/commands, which are then used interactively within an agent session

## 3. Non-Goals

- **General Purpose Language**: Not intended for general computation, complex logic, or defining algorithms directly
- **Complex Data Structures**: Primarily focused on text, code blocks, simple key-value pairs, and lists implied by structure, not deeply nested arbitrary data like full JSON/YAML
- **Runtime Definition**: Does not specify how commands are executed, only how they are represented and parameterized

## 4. Syntax Specification

### Basic Rules
- **Encoding**: Files should be UTF-8 encoded
- **Structure**: Primarily line-based. Hierarchy is defined by indentation
- **Indentation**: Spaces must be used for indentation (tabs are disallowed). Consistent indentation levels (e.g., 2 or 4 spaces per level) must be used within a file
- **Keywords**: All primary language constructs (Directives and Commands) start with a dot (.)
- **Directives**: Use .PascalCase (e.g., .Instructions, .Context). They define the sections and metadata of the GaryML document
- **Commands**: Use .camelCase (e.g., .refactor, .applyPatch). They represent actions to be performed by the agent, typically within a .Execute block
- **Comments**: Lines beginning with # are comments and should be ignored by parsers

### Parameters
Key-value pairs are specified as `key: value` on lines indented under a Directive or Command.

Keys are typically snake_case or camelCase identifiers.

#### Value Types
- **Strings**: Can be unquoted if they don't contain special characters or leading/trailing whitespace. Use single (') or double (") quotes if needed
- **Numbers**: Standard integer or floating-point notation
- **Booleans**: true or false
- **Lists**: Can be represented by repeating the key on subsequent lines with the same indentation
- **References**: Use @id to refer to another block (e.g., a .Context or .Diff block) defined with an id parameter

### Text and Code Blocks

#### Text Blocks
Multi-line strings are enclosed in triple double quotes ("""). Indentation within the block relative to the opening """ is preserved.

Example:
```garyml
.Instructions
  """
  This is the first line.
    This line is indented.
  This is the last line.
  """
```

#### Code Blocks
Fenced code blocks using triple backticks (```) are used for code snippets, typically within .Context or .Diff directives. An optional language identifier can follow the opening backticks.

Example:
```garyml
.Context id=my_code type=file path=main.go
  ```go
  package main

  func main() {
      println("Hello")
  }
  ```
```
```
## 5. Core Directives (.PascalCase)

- **.GaryML**: (Optional) Root element. Can have version: x.y
- **.Role**: Defines the persona the agent should adopt. Content is a string
- **.Instructions**: Main task description, rules. Content is typically a Text Block
- **.Context**: Provides background data/files
  - Required: id: identifier parameter for referencing
  - Optional: path: string, type: string (e.g., file, schema, text), format: string (e.g., python, json)
  - Content is a Code Block or Text Block
- **.Plan**: Outlines steps. Content is typically a Text Block
- **.Execute**: Contains one or more .camelCase commands to be run
- **.Query**: Specific question/sub-task for the agent. Content is a string
- **.Examples**: Provides few-shot examples. Can contain nested GaryML directives/commands
- **.Diff**: Embeds a code difference
  - Required: id: identifier
  - Required: format: string (e.g., v4a, search_replace, unified)
  - Content is a Text Block containing the diff
- **.Documentation**: Wraps documentation content. Optional title: string
- **.Section**: Groups items within .Documentation. Requires title: string
- **.OutputFormat**: Specifies desired response structure. Content is typically a Text Block
- **.ReasoningStrategy**: Guides the agent's thinking process. Content is typically a Text Block

## 6. Executable Commands (.camelCase)

Commands represent actions for the agent (e.g., .refactor, .test, .applyPatch, .help, .log). They are:
- Defined by their .camelCase name
- Can accept key: value parameters indented underneath
- Available commands may be built-in or defined within project's GaryML file using .Documentation / .Section

## 7. Parsing Considerations

### Key Implementation Aspects
- **Line-Based Processing**: Read the file line by line
- **Indentation Tracking**: Maintain a stack or counter to track the current indentation level
- **Keyword Recognition**: Use regular expressions to identify lines starting with . followed by PascalCase or camelCase identifiers
- **Parameter Parsing**: Identify key: value pairs based on indentation and the presence of :
- **Block Handling**: Implement a state machine to detect the start (""" or ```) and end of multi-line blocks
- **Output Structure**: Produce an Abstract Syntax Tree (AST) representing the hierarchical structure

### Parser Configuration
- **Indentation**: Define expected number of spaces per indent level
- **Keyword Regex**: `^\s*(\.)([A-Z]\w*|[a-z]\w*)`
- **Parameter Regex**: `^\s*([\w_]+)\s*:\s*(.*)$`
- **Block Delimiters**: """ and ```
- **AST Node Types**: Define data structures for different nodes (GaryMLDocument, DirectiveNode, etc.)

## 8. Example Usage

```garyml
.GaryML version: 1.0
.Role: "Go Scaffolder"
.Instructions
  """Generate basic Go HTTP handler..."""
.Plan
  """1. Determine packages..."""
.Execute
  .scaffold
    language: go
    output_path: pkg/handlers/health.go
.OutputFormat
  """Confirm file creation..."""
```

## 9. Future Considerations

- Error handling syntax
- More complex list/map syntax
- Include/import mechanism for splitting large definitions
- Schema definition/validation for GaryML files
