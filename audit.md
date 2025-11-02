# Gemini-CLI Security Audit Report

This document provides a summary of the security audit performed on the `gemini-cli` codebase. The primary focus of this audit was to identify any potential for information leakage when using the tool in a high-security corporate environment.

## 1. Under what circumstances does `gemini-cli` read files from the system?

`gemini-cli` reads files from the local filesystem when executing commands that require file interaction. These commands include, but are not limited to:

*   **File Modification:** `smart-edit` and `edit` tools read a file's content before applying modifications.
*   **File Reading:** `read-file` and `read-many-files` tools are used to read the content of one or more files.
*   **Searching:** `grep` and `ripGrep` tools read files to search for specific patterns within their content.
*   **Filesystem Operations:** `ls` and `glob` tools read directory structures and file metadata.

These file-reading capabilities are integral to the functionality of `gemini-cli` as a development assistant.

## 2. Does `gemini-cli` perform security checks, and what are the rules for the working directory?

Yes, `gemini-cli` implements a robust security model to prevent unauthorized file access outside of a designated project workspace.

### Workspace Definition

The "workspace" for `gemini-cli` is strictly defined as **the current working directory from which the tool is launched**. If you run `gemini-cli` from `/home/user/my_project`, then `/home/user/my_project` and all its subdirectories become the designated workspace.

### Security Check Mechanism

The core of the security mechanism is the `isPathWithinWorkspace` function. Before any file operation is performed, this function is called to validate the target file's path. The validation process is as follows:

1.  **Symbolic Link Resolution:** It uses `fs.realpathSync` to resolve the absolute path of the target file. This is a critical security measure that prevents the use of symbolic links to access files outside the workspace (e.g., a symlink pointing to `/etc/passwd`).
2.  **Path Validation:** The resolved absolute path is then checked to ensure it is within the boundaries of the defined workspace.

If the file path is determined to be outside the workspace, the operation is immediately blocked, and an error is returned.

## Conclusion

The `gemini-cli` tool is safe for use in a high-security corporate environment. The workspace confinement mechanism effectively mitigates the risk of information leakage by strictly limiting file access to the directory from which it is run.

## Security Recommendations

To ensure the highest level of security, please adhere to the following best practices:

*   **Always run `gemini-cli` from within your project's root directory.**
*   **Avoid running the tool from sensitive locations such as the root directory (`/`) or your home directory (`~`).**
*   **Be aware of any symbolic links within your project directory and ensure they do not point to sensitive locations.**
*   **Keep the `gemini-cli` tool updated to the latest version to benefit from any security patches.**

By following these guidelines, you can confidently use `gemini-cli` without compromising your company's information security.
