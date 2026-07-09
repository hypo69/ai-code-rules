# ENGINEERING STANDARD v1.0 (FastAPI Foundry Edition)
**Previous Version:** v0.3.1  
**Status:** ✅ Production Ready  
**Copyright:** © 2026 hypo69  

---

## 📋 Table of Contents
1. [Introduction and Scope](#1-introduction-and-scope)
2. [Normative Keywords (RFC 2119)](#2-normative-keywords-rfc-2119)
3. [Fundamental Architectural Principles](#3-fundamental-architectural-principles)
4. [General Design Rules and Code Smells Mitigation](#4-general-design-rules-and-code-smells-mitigation)
5. [Commenting and Documentation Standards](#5-commenting-and-documentation-standards)
6. [Language Style Guides](#6-language-style-guides)
    * [6.1 Python Style Guide (v3.12+)](#61-python-style-guide-v312)
    * [6.2 PHP & WordPress Style Guide (v8.3+)](#62-php--wordpress-style-guide-v83)
    * [6.3 JavaScript & TypeScript Style Guide (ES2024)](#63-javascript--typescript-style-guide-es2024)
    * [6.4 HTML & CSS/SCSS Style Guide](#64-html--cssscss-style-guide)
7. [Configuration, File Operations, and Logging](#7-configuration-file-operations-and-logging)
8. [AI Coding Rules](#8-ai-coding-rules)
9. [Appendices and Templates](#9-appendices-and-templates)

---

## 1. Introduction and Scope

This document establishes the **Unified Engineering Standard** (Engineering Standard v1.0) for the intelligent assistant ecosystem. The standard is designed as a governing technical regulation for human developers and as a system instruction for AI assistants (LLMs) participating in the writing, refactoring, and auditing of the codebase.

### 1.1 Scope
The requirements of this standard apply to:
* All new and existing source files across all project repositories.
* Architectural decisions, directory structures, configuration storage formats, and secrets management.
* Code formatting, comment structures, and automatic documentation generation.

---

## 2. Normative Keywords (RFC 2119)

To ensure unambiguous interpretation of requirements, this standard employs the compliance terminology defined in RFC 2119:

* **MUST**: This word means that the definition is an absolute requirement of the specification. Failure to comply with this requirement is a critical error.
* **MUST NOT**: This phrase means that the definition is an absolute prohibition of the specification.
* **SHOULD**: This word means that there may exist valid reasons in particular circumstances to ignore a particular rule, but the developer (or AI) MUST fully understand and carefully weigh the implications of choosing a different path and document the justification.
* **SHOULD NOT**: This phrase means that the particular behavior is not recommended, requiring a detailed technical justification when implemented.
* **MAY**: This word means that an item is truly optional.

---

## 3. Fundamental Architectural Principles

All technical decisions within the project MUST align with the following high-priority principles:

### 3.1 Readability Over Cleverness
Code is written for subsequent reading by humans or analysis by AI. Using hidden syntax tricks that reduce readability MUST NOT be permitted merely to save a few lines of code.

### 3.2 Single Responsibility Principle (SRP)
Every module, class, and function MUST solve only one logical task. If a function performs several unrelated actions, it SHOULD be decomposed.

### 3.3 Explicit is Better than Implicit
* Dependency passing MUST be performed explicitly (Dependency Injection).
* Global state within the codebase MUST NOT be used; all shared parameters MUST be passed through a controlled configuration object `Config` (or a similar singleton context).

### 3.4 Early Return & Fail-Fast
Functions MUST terminate execution immediately upon detecting invalid input parameters or failing preconditions.

```python
# ✅ Recommended Format (Early Return)
def process_user_data(data: dict) -> bool:
    if not data:
         return False
    # Execution of the main logic
    return True

# ❌ Non-Recommended Format (Deep Nesting)
def process_user_data(data: dict) -> bool:
    if data:
         # Execution of the main logic
         return True
    return False
```

---

## 4. General Design Rules and Code Smells Mitigation

To maintain the quality of the architecture, the following structural complexity limits are established:

### 4.1 Nesting Limit
The nesting depth of loops and conditional statements inside a single function SHOULD NOT exceed **3 levels**. If nesting is deeper, it is a clear signal for functional decomposition.

### 4.2 Avoid Duplication (DRY)
Duplication of similar logic across multiple modules MUST NOT be permitted. Identical operations (e.g., API requests to OpenAI-compatible endpoints of various providers) MUST be unified into generic connectors parametrized through configuration.

### 4.3 Dead Code
Unused variables, imports, functions, and dead files MUST be deleted immediately after refactoring. Retaining commented-out code "for the future" MUST NOT be permitted.

---

## 5. Commenting and Documentation Standards

### 5.1 Philosophy of Comments: "Why", Not "What"
Comments in the code SHOULD explain the engineering decisions and logical premises behind choosing a specific implementation path, rather than duplicating the syntax of the language.

```javascript
// ✅ Correct: explanation of structural choices and performance reasoning
// Usage of Set instead of Array to ensure unique IDs in O(1) complexity
const activeConnections = new Set();

// ❌ Incorrect: redundant syntax description
// Creation of a new set of active connections
const activeConnections = new Set();
```

### 5.2 Use of Deverbal Nouns in Comments
When writing comments in Russian (permitted for Python and PowerShell), instead of verbs or verbal participles, **deverbal nouns** (отглагольные существительные) expressing a process or state MUST be used.

* **Bad:** `// Отправляет запрос`, `// Суммаризирует текст`, `// Объединяет данные`
* **Good:** `// Отправка запроса`, `// Суммаризация текста`, `// Объединение данных`

For English comments (mandatory for the entire Web stack and PHP; recommended for others), nouns or gerunds expressing a process SHOULD be used: e.g., `Initialization`, `Verification`, `Loading`, `Execution` (instead of verbs like `Initializes`, `Verifies`, `Loads`).

### 5.3 Commenting Language Zones
Language restrictions in comments are established to prevent Unicode breakage, file encoding corruption, and critical interpretation errors in web and PHP environments.

| File Extensions | Code Comments | Documentation (Docstrings / JSDoc) | File Headers (Header Blocks) | Reason for Restriction |
| :--- | :--- | :--- | :--- | :--- |
| `.js`, `.ts`, `.jsx`, `.tsx` | **English only** | **English only** | **English only** | **MUST NOT** contain Cyrillic characters to prevent Unicode corruption in web environments. |
| `.css`, `.scss`, `.sass` | **English only** | **English only** | **English only** | **MUST NOT** contain Cyrillic characters to prevent Unicode corruption during asset compilation. |
| `.html` | **English only** | — | **English only** | **MUST NOT** contain Cyrillic characters to prevent encoding issues during page rendering. |
| `.php` | **English only** | **English only** | **English only** | **MUST NOT** contain Cyrillic characters to prevent parsing errors and Unicode corruption in PHP environments. |
| `.py`, `.ps1` | **Russian / English** | **Russian / English** | **Russian / English** | Cyrillic characters are allowed (strictly UTF-8 without BOM). |

### 5.4 Function Documentation Standard (`hypo69 docblock`)
A single unified section structure MUST be used for all public methods, functions, and classes across all programming languages.

#### Exclusion of Sphinx and reStructuredText
Using Sphinx/reST constructs (such as `.. module::`, `:platform:`, `:synopsis:`, `:param:`, `:returns:`) MUST NOT be permitted. If such blocks are found in existing code, they MUST be removed and replaced with the standard format.

#### Section Structure
The order and naming of sections in Docstrings/Docblocks MUST strictly correspond to the following format:

```text
Short Description        ← Single line summarizing the entity
Long Description         ← (Optional) Detailed explanation of the implementation choices

Args:
    param_name (type): Parameter description, constraints, and default values.

Returns:
    type: Description of the return value and return conditions.

Exceptions:
    ErrorType: Conditions under which the exception is thrown.

Examples:
    An executable example of the function invocation.
```

* The sections `Args:`, `Returns:`, `Exceptions:`, and `Examples:` — MUST be present for every public function. If a section is not applicable (e.g., no exceptions or parameters), it is omitted.
* The order of sections MUST NOT be altered.
* The term `Params:` — PROHIBITED; only `Args:` MUST be used.
* The term `Raises:` — PROHIBITED; only `Exceptions:` MUST be used.
* The `Examples:` section MUST contain a minimal working code example that can be copied and tested directly.

---

## 6. Language Style Guides

### 6.1 Python Style Guide (v3.12+)

Development in Python MUST utilize the modern features of versions 3.12+ (use of `pathlib`, `enum.StrEnum`, `match`, `@override`, `typing.Self`, `Protocol`, `TypedDict`).

#### Python File Header
Every Python file MUST begin with the following structured header:

```python
# -*- coding: utf-8 -*-
# =============================================================================
# Process Name: Integration with Local Foundry Server
# =============================================================================
# Description:
#   Managing interaction with Foundry API via HTTP protocol.
#   Verifying port availability and process management.
#
# Examples:
#   >>> from src.foundry import FoundryConnector
#   >>> connector = FoundryConnector(port=3000)
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

#### Grouping of Imports
Imports MUST be ordered alphabetically and separated into three distinct logical blocks with an empty line:
1. Standard library imports (stdlib).
2. Third-party library imports.
3. Local project imports, starting from the root import.

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

#### Python Class and Function Implementation Example
```python
from typing import Self, Optional

class FoundryConnector:
    """Establishment of connection with the Foundry API.

    Management of local processes and monitoring of port state.

    Attributes:
        port (int): Network port of the server.
        status (str): Current connection state.
    """

    def __init__(self, port: int) -> None:
        """Initialization of the connection object."""
        self.port: int = port
        self.status: str = "disconnected"

    def execute_connection(self, timeout: Optional[int] = None) -> Self:
        """Initiation of the connection process to the server.

        Args:
            timeout (Optional[int]): Maximum waiting time in seconds.
                                     Default value: None.

        Returns:
            Self: Current instance for supporting method chaining.

        Exceptions:
            ConnectionError: Failure during socket initialization when the port is unavailable.

        Examples:
            >>> connector = FoundryConnector(port=3000)
            >>> connector.execute_connection(timeout=10)
            <FoundryConnector object at 0x...>
        """
        # Verification of port initialization before connection startup
        if self.port <= 0:
            raise ConnectionError("Invalid port number")

        self.status = "connected"
        return self
```

---

### 6.2 PHP & WordPress Style Guide (v8.3+)

PHP development is aligned with modern PHP 8.3+ and WordPress development standards.
* The use of deprecated code is prohibited; all plugin logic SHOULD be encapsulated within Singleton classes.
* Use of nonce verifications, sanitization, and output escaping is mandatory.
* **Cyrillic characters are completely prohibited** in the source code of PHP files (including headers and comments) to prevent Unicode breakage.

#### PHP File Header
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

When writing scripts, ES2024 standard MUST be followed (asynchrony via `async/await`, destructuring, optional chaining `?.`, nullish coalescing `??`).
* **Cyrillic characters are completely prohibited** in the source code of JS/TS files (including headers and comments) to prevent Unicode breakage.

#### JS/TS File Header
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
 *   const client = new ApiClient('https://api.openai.com/v1');
 *   await client.fetchData('/chat/completions', { model: 'gpt-4o' });
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
     *   const client = new ApiClient('https://api.example.com');
     *   const response = await client.postData('/submit', { id: 10 });
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

* **Cyrillic characters are completely prohibited** in the source code of HTML/CSS/SCSS files (including headers and comments) to prevent Unicode breakage.

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

## 7. Configuration, File Operations, and Logging

### 7.1 Input/Output Operations (Files and JSON)
All file operations in Python MUST be executed exclusively through specialized wrapper functions from the project's utility library:
* **Reading data:** `j_loads()`, `j_loads_ns()`, `read_text_file()`.
* **Writing data:** `j_dumps()`, `save_text_file()`.

#### Utility Features:
These functions automatically perform I/O error logging and create the missing directory structure (when writing).

#### Rule for Processing Results:
Calls to file utilities **MUST NOT** be wrapped in `try/except` blocks. Instead, execution results MUST be verified explicitly in the code by checking the returned data:

```python
# ✅ Correct: Explicit checking of the return value without redundant error catching
config_data = j_loads(config_path)
if not config_data:
    logger.error(f"Failed to load configuration file: {config_path}")
    return None

# ❌ Incorrect: Redundant wrapping of a system function in try/except
try:
    config_data = j_loads(config_path)
except Exception as e:
    logger.error("File system error occurred", e)
```

### 7.2 Separation of Configurations and Secrets
Application parameters are divided into two categories: public settings and private settings (secrets).

| Category | Storage Method | Distribution Mechanism | Example |
| :--- | :--- | :--- | :--- |
| **Public** | `config.json` | Stored in the Git repository | Ports, timeouts, paths to RAG indexes |
| **Secret** | `.env` | **MUST NOT** be committed to Git (added to `.gitignore`) | API keys, JWT secrets, DB passwords |

#### Use of Environment Variables Inside JSON
Direct specifications of secret values in the public `config.json` file are prohibited. A strictly defined placeholder format MUST be used for referencing secrets: `"${ENVIRONMENT_VARIABLE_NAME}"`.

The system environment variable processor automatically substitutes values from `.env` in place of these placeholders during application startup.

```json
{
  "huggingface": {
    "api_endpoint": "https://api-inference.huggingface.co",
    "token": "${HUGGING_FACE_TOKEN}"
  }
}
```

#### Mandatory Environment Variables (.env.example)
When adding any new secret to the codebase, its placeholder MUST simultaneously be added to the `.env.example` file with a description of its purpose:

```env
# HuggingFace token for working with closed models
HUGGING_FACE_TOKEN=hf_your_token_here

# Network parameters of the local Foundry environment
FOUNDRY_BASE_URL=http://localhost
FOUNDRY_DYNAMIC_PORT=3000
```

### 7.3 Logging and Information Output
* Log output across all execution environments MUST be routed exclusively through the project's standard logger object `src.logger.logger`.
* The use of the system call `print()` for outputting information to the console **MUST NOT** be permitted in production code.
* For debug printing of complex structures to the console (in the development environment), the `pprint` pretty-print utility MUST be used.

```python
# ✅ Logging an error with an exception
logger.error("API connector initialization failure", ex, exc_info=True)

# ✅ Printing a debugging structure
pprint(debug_data_structure)
```

---

## 8. AI Coding Rules

AI assistants working with the project's codebase ARE OBLIGATED to strictly follow these protocols for code analysis and modification:

### 8.1 Pre-Analysis Protocol
1. Before modifying a file, the AI **MUST** read the file in its entirety and analyze its imports structure, comments, and architectural relationships.
2. The AI **MUST** align the proposed changes with the general architectural principles of the project (Early Return, DRY, Single Responsibility).

### 8.2 Modification and Integrity Protocol
1. **No Refactoring for the Sake of Refactoring:** The AI **MUST NOT** rewrite correctly functioning code merely to alter visual style, unless the current code violates this engineering standard.
2. **Preservation of Existing API:** When making changes, the AI **MUST** preserve the signatures of public methods, classes, and backward-compatibility interfaces, unless otherwise explicitly stated in the task.
3. **Prohibition of Incomplete Blocks:** Generating placeholder comments, such as `// TODO: implement later`, `/* FIXME */`, or inserting empty stub structures instead of writing concrete logic **MUST NOT** be permitted.
4. **Preservation of System Comments:** Debugging symbols (lines with ellipses `...`), as well as auto-generated headers containing file metadata, **MUST** be carefully preserved in their original state.

---

## 9. Appendices and Templates

### 9.1 Template for README.md in Project Directories
A `README.md` file describing its structure and functional purpose **MUST** reside within every directory of the project. The file MUST be constructed according to the following strict template:

```markdown
# Directory Name (e.g., src/utils)

## Description
A brief description of the purpose of this folder and its role in the global project architecture.

## File and Module Overview
* `printer.py` — Pretty printing of complex data structures to the console.
* `json_processor.py` — Safe reading, parsing, and writing utilities for JSON files.

## Dependencies and Relations
* The module is imported across all application levels to facilitate utility operations.
* Depends on system data serialization utilities.
```

### 9.2 Quality Control Commit Checklist
Before completing a task and committing changes to the repository, the developer or AI assistant **MUST** perform a verification according to the following checklist:

* [ ] Every modified or created source file contains a completed, standard file header (Header Block) matching its programming language.
* [ ] All created public methods, classes, and functions contain a detailed Docstring/Docblock in the `hypo69 docblock` format (Args, Returns, Exceptions, Examples).
* [ ] Any Sphinx/reStructuredText constructs are entirely excluded from all sections of the documentation.
* [ ] Comment language zones are strictly observed (Web and PHP — English only; Python — Russian/English).
* [ ] In Russian comments, verbs are replaced by deverbal nouns (e.g., "Инициализация подключения" instead of "Инициализирует подключение").
* [ ] In English comments for Web and PHP, only nouns or gerunds are used (`Initialization`, `Verification`, `Loading`, `Execution`).
* [ ] Cyrillic characters are completely absent in JS, PHP, HTML, and CSS files to prevent Unicode breakage.
* [ ] The code contains no `print()` calls; all outputs are routed through `logger` or `pprint`.
* [ ] All new secret parameters are moved to `.env` (and their placeholders are added to `.env.example`), and references to them in `config.json` are formatted as `"${VAR_NAME}"`.
* [ ] File read and write operations for JSON/text are executed via wrappers (`j_loads`, `save_text_file`, etc.) with explicit result validation and without redundant `try/except` blocks.
* [ ] Unused temporary files, dead code, and commented-out logic blocks are entirely removed. In altered directories, `README.md` files are updated or created.
