# ENGINEERING STANDARD v1.0 (FastAPI Foundry Edition)
**Previous Version:** v0.3.1  
**Status:** ✅ Production Ready  
**Copyright:** © 2026 hypo69  

---

## 📋 Table of Contents
1. [Introduction and Scope](#1-introduction-and-scope)
2. [Normative Keywords (RFC 2119)](#2-normative-keywords-rfc-2119)
3. [Fundamental Architectural Principles](#3-fundamental-architectural-principles)
4. [General Design Rules and Code Smells Elimination](#4-general-design-rules-and-code-smells-elimination)
5. [Commenting and Documentation Standards](#5-commenting-and-documentation-standards)
6. [Language Development Standards](#6-language-development-standards)
    * [6.1 Python Style Guide (v3.12+)](#61-python-style-guide-v312)
    * [6.2 PHP & WordPress Style Guide (v8.3+)](#62-php--wordpress-style-guide-v83)
    * [6.3 JavaScript & TypeScript Style Guide (ES2024)](#63-javascript--typescript-style-guide-es2024)
    * [6.4 HTML & CSS/SCSS Style Guide](#64-html--cssscss-style-guide)
7. [Working with Configuration, Files, and Logging](#7-working-with-configuration-files-and-logging)
8. [AI Coding Rules](#8-ai-coding-rules)
9. [Appendices and Templates](#9-appendices-and-templates)

---

## 1. Introduction and Scope

This document defines the **Unified Engineering Standard** (Engineering Standard v1.0) for the intelligent assistant ecosystem. The standard is designed as a guiding technical regulation for developers and as a system instruction for AI assistants (LLMs) involved in writing, refactoring, and auditing the codebase.

### 1.1 Scope
The standard applies to:
* All new and existing source files in all project repositories.
* Architectural decisions, directory structures, and formats for configuration and secret storage.
* Code formatting, comment structures, and auto-documentation generation.

---

## 2. Normative Keywords (RFC 2119)

To ensure unambiguous interpretation of requirements, the following keywords are used in accordance with the RFC 2119 specification:

* **MUST (REQUIRED)**: An absolute requirement for implementation. Failure to comply makes the codebase unacceptable for integration.
* **MUST NOT (PROHIBITED)**: An absolute prohibition on using a construct, method, or pattern.
* **SHOULD (RECOMMENDED)**: Means that there may be valid reasons for deviating from the rule, but the developer (or AI) must fully evaluate the consequences, document the reasoning, and weigh the risks.
* **SHOULD NOT (NOT RECOMMENDED)**: Means the practice is undesirable and requires detailed technical justification if implemented.
* **MAY (OPTIONAL)**: A completely optional action or tool choice.

---

## 3. Fundamental Architectural Principles

All technical decisions within the project **MUST** comply with the following priority principles:

### 3.1 Readability Over Cleverness
Code is written for subsequent reading by humans or AI analysis. Use of hidden syntax tricks that degrade readability **MUST NOT** be permitted for the sake of saving lines of code.

### 3.2 Single Responsibility Principle
Each module, class, and function **MUST** solve only one logical task. If a function performs multiple unrelated actions, it **SHOULD** be separated.

### 3.3 Explicit is Better than Implicit
* Dependency injection **MUST** be performed explicitly.
* Global state of the codebase **MUST NOT** be used; all shared parameters **MUST** be passed through a controlled `Config` object (or similar singleton context).

### 3.4 Early Return & Fail-Fast
Functions **MUST** terminate execution immediately upon detecting invalid input or failure to meet preconditions.

```python
# ✅ Recommended format (Early Return)
def process_user_data(data: dict) -> bool:
    if not data:
         return False
    # Main logic execution
    return True

# ❌ Undesirable format (Deep nesting)
def process_user_data(data: dict) -> bool:
    if data:
         # Main logic execution
         return True
    return False
```

### 3.5 Configuration over Hardcode
Hardcoded parameters in the application source code **MUST NOT** be used. All parameters, including network ports, URLs, timeouts, directory paths, limits, and operational modes, **MUST** be read from an external JSON configuration file (via the `config` object) or environment variables.

* **❌ Incorrect:**
  ```python
  port = 8000
  ```
* **✅ Correct:**
  ```python
  port = config.port
  ```

### 3.6 Categorical Prohibition of Using and Referencing `None`
Use of the keyword `None`, as well as any explicit or implicit references to it in the source code, is **STRICTLY PROHIBITED** (**MUST NOT**). Excluding this object is a fundamental requirement for type stability.

#### 3.6.1 Absolute Prohibition of `None` in Function Signatures (Default Parameters)
Function arguments **MUST NOT** accept `None` as a default value. Signatures **MUST** use empty default values of corresponding data types. `Optional[...]` is allowed only if the default value is strictly defined as a type other than `None` (e.g., `Optional[int] = 0`).

* **❌ Bad:**
  ```python
  def execute_connection(self, timeout: Optional[int] = None, input_str: Optional[str] = None) -> Self:
  ```
* **✅ Good:**
  ```python
  def execute_connection(self, timeout: Optional[int] = 0, input_str: Optional[str] = '') -> Self:
  ```

#### 3.6.2 Prohibition in Variable Initialization and Class Properties
Local, global variables, and class properties **MUST NOT** be initialized with `None`. Variables **MUST** be initialized only with empty values of corresponding types:
* Numeric types (`int`, `float`): `0` or `0.0`.
* String types (`str`): `''` (empty string).
* Boolean types (`bool`): `False`.
* Collections (`list`, `dict`): `[]` or `{}`.

* **❌ Bad:**
  ```python
  str_output = None
  dict_output = None
  ```
* **✅ Good:**
  ```python
  str_output = ''
  dict_output = {}
  ```

#### 3.6.3 Prohibition in Comparison Operators
Operators `is None` and `is not None` are **STRICTLY PROHIBITED** (**MUST NOT**). State validation **MUST** be performed via boolean truthiness or explicit validation of default values.

* **❌ Bad:**
  ```python
  if connection is None:
      ...
  ```
* **✅ Good:**
  ```python
  if not connection:
      ...
  ```

#### 3.6.4 Prohibition of `None` Returns and Implicit Returns
For the sake of eliminating ambiguity and improving type strictness, functions **MUST NOT** return `None` or use an empty `return` operator (which implicitly returns `None`).

Upon failure, lack of data, early exit, or error, functions **MUST** explicitly return `False` (or `false` for JS/PHP/HTML).

* **❌ Incorrect:**
  ```python
  return
  # or
  return None
  ```
* **✅ Correct:**
  ```python
  return False
  ```

---

## 4. General Design Rules and Code Smells Elimination

To maintain architectural quality, the following structural complexity limits are enforced:

### 4.1 Nesting Limit
Nesting depth of loops and conditionals within a single function **SHOULD NOT** exceed **3 levels**. If nesting goes deeper, it is a clear indicator that the function needs to be decomposed.

### 4.2 DRY (Don't Repeat Yourself)
Duplicate logic **MUST NOT** be permitted across modules. Identical operations (e.g., requesting OpenAI-compatible APIs across different providers) **MUST** be unified into parameterized, general-purpose connectors.

### 4.3 Dead Code
Unused variables, imports, and functions **MUST** be removed immediately after refactoring. Leaving commented-out code "for the future" **MUST NOT** be permitted.

### 4.4 Function Size Limit
The physical size of any function (excluding comments/docstrings) **MUST NOT** exceed **300 lines of code**. If the limit is exceeded, the function **MUST** be decomposed. The process must be split into multiple isolated, easily testable helper functions, while the main function serves as a dispatcher coordinating them according to the workflow scenario.

---

## 5. Commenting and Documentation Standards

### 5.1 Comment Philosophy: "Why" over "What"
Comments in the code **SHOULD** explain the engineering rationale rather than duplicate code syntax.

```javascript
// ✅ Correct: explanation of structural choices and performance reasoning
// Usage of Set instead of Array to ensure unique IDs in O(1) complexity
const activeConnections = new Set();

// ❌ Incorrect: redundant syntax description
// Creation of a new set of active connections
const activeConnections = new Set();
```

### 5.2 Use of Verbal Nouns in Comments
When writing comments in Russian (permitted for Python and PowerShell), instead of verb forms (verbs, participles), developers **MUST** use **verbal nouns** that express a process or state.

* **Bad:** `// Отправляет запрос` (Sends request), `// Суммаризирует текст` (Summarizes text)
* **Good:** `// Отправка запроса` (Request sending), `// Суммаризация текста` (Text summarization)

For English comments (web stack and PHP), developers **SHOULD** use Nouns or Gerunds that express a process: `Initialization`, `Verification`, `Loading`, `Execution` (instead of verbs like `Initializes`, `Verifies`, `Loads`).

### 5.3 Commenting Language Zones
Commenting language restrictions are enforced to prevent Unicode breakage, files corruption, and parsing errors in web environments and PHP.

| File Extensions | Comments in Code | Documentation (Docstrings / JSDoc) | File Headers (Header Blocks) | Restriction Rationale |
| :--- | :--- | :--- | :--- | :--- |
| `.js`, `.ts`, `.jsx`, `.tsx` | **English only** | **English only** | **English only** | **MUST NOT** contain Cyrillic to avoid Unicode breakage in web environments. |
| `.css`, `.scss`, `.sass` | **English only** | **English only** | **English only** | **MUST NOT** contain Cyrillic to avoid Unicode breakage during asset compilation. |
| `.html` | **English only** | — | **English only** | **MUST NOT** contain Cyrillic to prevent encoding issues during page rendering. |
| `.php` | **English only** | **English only** | **English only** | **MUST NOT** contain Cyrillic to prevent parsing errors and Unicode breakage in PHP runtime. |
| `.py`, `.ps1` | **Russian / English** | **Russian / English** | **Russian / English** | Cyrillic is permitted (strictly UTF-8 without BOM). |

### 5.4 Standard for Documenting Functions (`hypo69 docblock`)
For all public methods, functions, and classes across all languages, developers **MUST** use a single unified section format.

#### Sphinx and reStructuredText Exclusion
Use of Sphinx/reST keywords (such as `.. module::`, `:platform:`, `:synopsis:`, `:param:`, `:returns:`) **MUST NOT** be permitted. If such blocks are found in existing code, they **MUST** be removed and replaced with the standard format.

#### Structure of Sections
The order and naming of sections in a Docstring/Docblock **MUST** strictly follow this schema:

```text
Short Description        ← One-line description of the entity's purpose
Long Description         ← (Optional) Detailed explanation of implementation choices

Args:
    param_name (type): Parameter description, constraints, and defaults.

Returns:
    type: Description of the return value and conditions of return.

Exceptions:
    ErrorType: Conditions under which the exception is thrown.

Examples:
    An example call shown as executable code.
```

* Sections `Args:`, `Returns:`, `Exceptions:`, `Examples:` — **MUST** be present for every public function. If a section is not applicable (e.g., no exceptions), it is omitted.
* Section order **MUST NOT** be altered.
* The word `Params:` — PROHIBITED; only `Args:` is used.
* The word `Raises:` — PROHIBITED; only `Exceptions:` is used.
* The `Examples:` section **MUST** contain a minimal working example that can be copied and tested.

---

## 6. Language Development Standards

### 6.1 Python Style Guide (v3.12+)

Development in Python **MUST** use modern features of versions 3.12+ (use of `pathlib`, `enum.StrEnum`, `match`, `@override`, `typing.Self`, `Protocol`, `TypedDict`).

#### Python File Header Block Template
Every Python file **MUST** begin with the following structured header:

```python
# -*- coding: utf-8 -*-
# =============================================================================
# Process Name: Foundry Local Server Integration
# =============================================================================
# Description:
#   Foundry API interaction management via HTTP protocol.
#   Port availability checks and process management execution.
#
# Examples:
#   >>> from src.foundry import FoundryConnector
#   >>> connector = FoundryConnector(port=config.port)
#   >>> connector.verify_status()
#
# File: foundry_connector.py
# Project: Our Intelligent Assistant
# Package: FoundryIntegration
# Module: Core
# Class: FoundryConnector
# Function: verify_status
# Author: hypo69
# Copyright: © 2026 hypo69
# =============================================================================
```

#### Import Grouping
Imports **MUST** be sorted alphabetically and separated into three clear logical blocks with an empty line:
1. Standard libraries (stdlib).
2. Third-party libraries.
3. Current project imports, starting with the root import.

```python
import os
import sys
from pathlib import Path

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

from header import __root__
from src import gs
from src.logger import logger
from src.utils.printer import pprint
```

#### Class and Function Example in Python
```python
from typing import Self, Optional

class FoundryConnector:
    """Foundry API connection management.

    Local process control and network port monitoring.

    Attributes:
        port (int): Server network port number.
        status (str): Current connection state.
    """

    def __init__(self, port: int) -> None:
        """Connection object initialization."""
        self.port: int = port
        self.status: str = "disconnected"

    def execute_connection(self, timeout: Optional[int] = 0, input_str: Optional[str] = '') -> Self:
        """Connection process startup.

        Args:
            timeout (Optional[int]): Max waiting time in seconds.
                                     Default value: 0.
            input_str (Optional[str]): Input initialization string.
                                       Default value: ''.

        Returns:
            Self: Current instance for method chaining.

        Exceptions:
            ConnectionError: Socket initialization failure when port is unavailable.

        Examples:
            >>> connector = FoundryConnector(port=config.port)
            >>> connector.execute_connection(timeout=10, input_str='init')
            <FoundryConnector object at 0x...>
        """
        # Port initialization check before connection startup
        if self.port <= 0:
            raise ConnectionError("Invalid port number")

        self.status = "connected"
        return self
```

---

### 6.2 PHP & WordPress Style Guide (v8.3+)

Development in PHP is focused on PHP 8.3+ and WordPress development standards.
* Legacy code is prohibited; all plugin logic **SHOULD** be packaged into Singleton classes.
* Nonce verification, sanitization, and output escaping are mandatory.
* **Cyrillic is completely prohibited** in PHP source code (including headers and comments) to prevent Unicode corruption.

#### PHP File Header Block Template
```php
<?php
# -*- coding: utf-8 -*-
# =============================================================================
# Process Name: WordPress Post Metadata Processing
# =============================================================================
# Description:
#   Registration of custom metadata fields for plugin post types.
#   Sanitization of incoming requests and escaping output in templates.
#
# Examples:
#   1. Hook registration:
#        add_action('save_post', [Post_Metadata::get_instance(), 'save_meta']);
#
# File: class-post-metadata.php
# Project: Our Intelligent Assistant
# Package: PluginMeta
# Class: Post_Metadata
# Author: hypo69
# Copyright: © 2026 hypo69
# =============================================================================
```

#### Secure PHP / WordPress Class Example
```php
<?php
defined( 'ABSPATH' ) || exit;

class Post_Metadata {
    /**
     * Singleton instance.
     *
     * @var Post_Metadata|null
     */
    private static ?Post_Metadata $instance = null;

    /**
     * Instance retrieval.
     *
     * @return Post_Metadata
     */
    public static function get_instance(): Post_Metadata {
        if ( null === self::$instance ) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    /**
     * Constructor.
     */
    private function __construct() {
        $this->init_hooks();
    }

    /**
     * Hook initialization.
     */
    private function init_hooks() {
        add_action( 'save_post', [ $this, 'save_custom_fields' ], 10, 2 );
    }

    /**
     * Saving of custom metadata fields on post save action.
     *
     * Args:
     *   post_id (int) — Current WordPress post ID.
     *   post (WP_Post) — Post object.
     *
     * Returns:
     *   bool — Verification success.
     *
     * Exceptions:
     *   RuntimeException — Thrown on security verification failure.
     *
     * Examples:
     *   $meta_manager = Post_Metadata::get_instance();
     *   // Triggered automatically via save_post action hooks
     */
    public function save_custom_fields( int $post_id, WP_Post $post ): bool {
        // Verification of security token (nonce) presence
        if ( ! isset( $_POST['custom_meta_nonce'] ) || 
             ! wp_verify_nonce( $_POST['custom_meta_nonce'], 'save_custom_meta' ) ) {
            return false;
        }

        // Capabilities authorization check
        if ( ! current_user_can( 'edit_post', $post_id ) ) {
            return false;
        }

        // Sanitization and storage of user input parameters
        if ( isset( $_POST['custom_field_value'] ) ) {
            $sanitized_value = sanitize_text_field( wp_unslash( $_POST['custom_field_value'] ) );
            update_post_meta( $post_id, '_custom_field_key', $sanitized_value );
        }

        return true;
    }
}
```

---

### 6.3 JavaScript & TypeScript Style Guide (ES2024)

When writing scripts, ES2024 standards **MUST** be used (asynchrony via `async/await`, destructuring, optional chaining `?.`, nullish coalescing `??`).
* **Cyrillic is completely prohibited** in JS/TS source files (including headers and comments) to prevent Unicode corruption.

#### JS/TS File Header Block Template
```javascript
/**
 * =============================================================================
 * Process Name: Asynchronous API Provider Interaction
 * =============================================================================
 * Description:
 *   Abstract client implementation for executing HTTP requests to OpenAI-compatible APIs.
 *   Provides dynamic authorization headers configuration based on provider parameters.
 *
 * Examples:
 *   const client = new ApiClient(config.openai.baseUrl);
 *   await client.fetchData('/chat/completions', { model: config.openai.model });
 *
 * File: api-client.js
 * Project: Our Intelligent Assistant
 * Module: Network
 * Class: ApiClient
 * Author: hypo69
 * Copyright: © 2026 hypo69
 * =============================================================================
 */
```

#### JS/TS Code Example
```javascript
class ApiClient {
    /**
     * Initialization of the base API client.
     *
     * @param {string} baseUrl - Base endpoint URL path.
     */
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
    }

    /**
     * Asynchronous HTTP POST execution to selected path.
     *
     * Args:
     *   endpoint (string) — API endpoint target path.
     *   payload (object) — Request body payload details.
     *
     * Returns:
     *   object — Resolved JSON data response object.
     *
     * Exceptions:
     *   Error — Thrown on network timeout or failed status codes.
     *
     * Examples:
     *   const client = new ApiClient(config.api.baseUrl);
     *   const response = await client.postData('/submit', { id: config.api.id });
     */
    async postData(endpoint, payload) {
        // Checking of endpoint parameter existence
        if (!endpoint) {
            throw new Error("Missing endpoint parameter");
        }

        const targetUrl = `${this.baseUrl}${endpoint}`;
        
        // Execution of fetch request
        const response = await fetch(targetUrl, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload ?? {})
        });

        if (!response.ok) {
            throw new Error(`HTTP Error Status: ${response.status}`);
        }

        return await response.json();
    }
}
```

---

### 6.4 HTML & CSS/SCSS Style Guide

* **Cyrillic is completely prohibited** in HTML/CSS/SCSS source files (including headers and comments) to prevent Unicode corruption.

#### HTML File Header Template
```html
<!--
===============================================================================
Process Name: Dashboard Settings Configuration Layout
===============================================================================
Description:
    Core template for managing system integrations, connection parameters, and API keys.
    Utilizes data-i18n attributes on label tags to ensure translation support.

File: dashboard-settings.html
Project: Our Intelligent Assistant
Module: AdminUI
Author: hypo69
Copyright: © 2026 hypo69
===============================================================================
-->
```

#### CSS/SCSS File Header Template
```css
/*
===============================================================================
Process Name: Dashboard Settings Styling Interface
===============================================================================
Description:
    Layout and visual rules for inputs, action forms, and administration panels.
    Implements CSS Grid layouts and responsive variables for system themes.

File: admin-styles.css
Project: Our Intelligent Assistant
Module: AdminUI
Author: hypo69
Copyright: © 2026 hypo69
===============================================================================
*/
```

---

## 7. Working with Configuration, Files, and Logging

### 7.1 File I/O Operations (Files and JSON)
All file operations in Python **MUST** be performed only via specialized wrappers from the project's utility library:
* **Reading data:** `j_loads()`, `j_loads_ns()`, `read_text_file()`.
* **Writing data:** `j_dumps()`, `save_text_file()`.

#### Utility Features:
These functions automatically perform error logging and handle folder creation (for write operations).

#### Result Handling Rule:
Since these utilities handle file system exceptions internally, log failures, and return a falsy value (`False`) on failure, **exceptions will not propagate** to the caller.

Wrapping wrapper utility calls in `try/except` blocks is redundant and **PROHIBITED** (**MUST NOT**). Because the utilities log failures automatically, the caller **MUST NOT** log them again (to prevent log duplication). The caller must only handle business logic and perform an early return.

```python
# ✅ Correct: Explicit return value verification without duplicate logging
# Note: j_loads logs the error internally on failure.
# The caller simply performs an early return without redundant logging.
config_data = j_loads(config_path)
if not config_data:
    return False

# ❌ Incorrect: Redundant logging and useless try/except
# Note: An exception will not be raised, and double logging spams logs.
try:
    config_data = j_loads(config_path)
    if not config_data:
        logger.error(f"Failed to load file")  # Log duplication!
        return False
except Exception as e:
    logger.error("FS Error", e)  # Dead code, this block will never execute
```

### 7.2 Separation of Configurations and Secrets
Application parameters are separated into public configuration and private secrets.

| Type | Storage Method | Distribution Mechanism | Example |
| :--- | :--- | :--- | :--- |
| **Public** | `config.json` | Stored in Git repository | Ports, timeouts, RAG paths |
| **Secret** | `.env` | **MUST NOT** be in Git (added to `.gitignore`) | API keys, JWT secrets, DB passwords |

#### Using Environment Variables inside JSON
Directly hardcoding secret values in `config.json` is prohibited. Placeholder references **MUST** be used in this format: `"${ENVIRONMENT_VARIABLE_NAME}"`.

The environment variable handler automatically replaces placeholders with `.env` values at startup.

```json
{
  "huggingface": {
    "api_endpoint": "https://api-inference.huggingface.co",
    "token": "${HUGGING_FACE_TOKEN}"
  }
}
```

#### Required Environment Variables (.env.example)
When adding any new secret, its placeholder **MUST** be simultaneously added to `.env.example` with a description:

```env
# HuggingFace token for restricted models access
HUGGING_FACE_TOKEN=hf_your_token_here

# Local Foundry network parameters
FOUNDRY_BASE_URL=http://localhost
FOUNDRY_DYNAMIC_PORT=3000
```

### 7.3 Logging and Output
* Log output across environments **MUST** be executed only via the standard project logger: `src.logger.logger`.
* Using `print()` **MUST NOT** be permitted in production code.
* For pretty-printing complex structures in development environments, the `pprint` utility **MUST** be used.

```python
# ✅ Exception logging
logger.error("API connector initialization failure", ex, exc_info=True)

# ✅ Debug print
pprint(debug_data_structure)
```

---

## 8. AI Coding Rules

AI assistants working with the project codebase are REQUIRED to strictly follow these protocols when analyzing and modifying files:

### 8.1 Preliminary Analysis Protocol
1. Before modifying a file, the AI **MUST** read the file in its entirety to analyze imports, comments, and architectural relationships.
2. The AI **MUST** align changes with the project's architectural principles (Early Return, DRY, Single Responsibility).

### 8.2 Modification and Integrity Preservation Protocol
1. **No refactoring for the sake of refactoring:** The AI **MUST NOT** rewrite working code to match style preferences unless the code violates this engineering standard.
2. **Preserve existing APIs:** The AI **MUST** preserve public method signatures, classes, and backward compatibility unless explicitly instructed otherwise.
3. **No incomplete blocks:** Generating marker comments like `// TODO: implement later` or empty placeholders instead of writing logic **MUST NOT** be permitted.
4. **Preserve system comments:** Ellipsis debug lines (`...`) and auto-generated file header metadata blocks **MUST** be preserved.

---

## 9. Appendices and Templates

### 9.1 README.md Template for Project Directories
Every directory **MUST** contain a `README.md` file describing its structure and purpose. The file is written according to this strict template:

```markdown
# Directory Name (e.g., src/utils)

## Description
A brief description of this folder's purpose and its role in the overall architecture.

## Files and Modules Overview
* `printer.py` — Pretty-printing of complex data structures in the console.
* `json_processor.py` — Safe reading, parsing, and writing utilities for JSON files.

## Dependencies and Relations
* Imported by all application layers for helper operations.
* Depends on system data serialization utilities.
```

### 9.2 Final Quality Control Checklist (Commit Checklist)
Before completing a task and committing, developers or AI assistants **MUST** verify their work against this checklist:

* [ ] Every created or modified file contains the standard header block for its programming language.
* [ ] All public methods, classes, and functions contain a `hypo69 docblock` (Args, Returns, Exceptions, Examples).
* [ ] Sphinx/reStructuredText keywords are excluded from documentation.
* [ ] Language commenting zones are observed (Web/PHP: English only; Python: Russian/English).
* [ ] Russian comments use verbal nouns (e.g., "Инициализация" instead of "Инициализирует").
* [ ] English web stack and PHP comments use Gerunds or Nouns (`Initialization`, `Execution`).
* [ ] JS, PHP, HTML, CSS files do not contain Cyrillic characters.
* [ ] No `print()` statements are in the code; logging uses `logger` or `pprint`.
* [ ] New secrets are in `.env` (with placeholders in `.env.example`), and references in `config.json` use `"${VAR_NAME}"`.
* [ ] I/O operations are handled via wrappers (`j_loads`, `save_text_file`) with falsy checks, without duplicate logging or redundant `try/except` blocks.
* [ ] No function or method returns `None` or uses an implicit empty `return`. Failure cases return `False` (or `false`).
* [ ] The keyword `None` and any references to it (defaults, initialization, comparisons) are completely excluded from the code. Variables use type-specific empty defaults (`0`, `''`, `[]`, `{}`).
* [ ] No parameters are hardcoded. Ports, timeouts, and URLs are loaded dynamically via `config` or environment variables.
* [ ] The physical size of any function body (excluding comments/docstrings) does not exceed 300 lines of code. Large functions are decomposed.
* [ ] No temporary files, dead code, or commented-out logic remain in directories. README.md files are updated.
