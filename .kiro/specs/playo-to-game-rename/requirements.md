# Requirements Document

## Introduction

The `game-ddd` workspace was originally built under the product name **Playo**. The product has been renamed to **Game**. This feature covers a comprehensive, verified rename of every occurrence of "Playo" (and its lowercase variant "playo") to "Game" (or "game") across all artefacts in the workspace: file names, Markdown documents, Mermaid diagram files, JSON analysis files, the Python inventory script, and — where technically feasible — the content of Excel workbooks.

The rename must be complete (no stale references left behind), consistent (casing rules applied uniformly), and safe (no broken cross-references, no data loss, no functional regressions in the Python script).

---

## Glossary

- **Rename_Tool**: The automated process (script or agent action) that performs find-and-replace and file-rename operations.
- **Text_File**: Any file whose content is human-readable text: `.md`, `.mmd`, `.json`, `.txt`, `.py`.
- **Excel_File**: Any `.xlsx` workbook in the workspace root (`Playo_DDD_v*.xlsx`).
- **Reference**: Any occurrence of the string `Playo` or `playo` (case-sensitive variants) inside a file's content or in a file's name.
- **Canonical_Mapping**: The agreed substitution table — `Playo` → `Game`, `playo` → `game`, `PLAYO` → `GAME` (if present).
- **Verification_Report**: A machine-generated summary confirming zero remaining References after the rename.
- **Cross_Reference**: A file path, hyperlink, or import statement in one file that points to another file by name.

---

## Requirements

### Requirement 1: Text File Content Rename

**User Story:** As a developer maintaining the `game-ddd` workspace, I want every occurrence of "Playo" and "playo" replaced with "Game" and "game" in all text files, so that the codebase consistently reflects the new product name.

#### Acceptance Criteria

1. WHEN the Rename_Tool processes a Text_File, THE Rename_Tool SHALL replace every occurrence of `Playo` with `Game` and every occurrence of `playo` with `game`, preserving all surrounding content unchanged.
2. THE Rename_Tool SHALL apply the Canonical_Mapping to all Text_Files in the workspace, including files in subdirectories (`analysis/excel_inventory/`, `scripts/`).
3. WHEN a Text_File contains zero References, THE Rename_Tool SHALL leave that file unmodified.
4. IF a Text_File cannot be read due to a permission or encoding error, THEN THE Rename_Tool SHALL log the file path and error reason without aborting the overall rename operation.
5. THE Rename_Tool SHALL preserve the original line endings (LF or CRLF) of each Text_File after replacement.

---

### Requirement 2: File Name Rename

**User Story:** As a developer, I want all files whose names contain "Playo" or "playo" renamed to use "Game" or "game", so that directory listings and tooling no longer reference the old product name.

#### Acceptance Criteria

1. WHEN the Rename_Tool identifies a file whose name contains `Playo`, THE Rename_Tool SHALL rename that file by substituting `Playo` with `Game` in the file name, keeping the extension and version suffix unchanged.
2. THE Rename_Tool SHALL rename all six Excel workbooks in the workspace root according to the Canonical_Mapping:
   - `Playo_DDD_v6.xlsx` → `Game_DDD_v6.xlsx`
   - `Playo_DDD_v6_1.xlsx` → `Game_DDD_v6_1.xlsx`
   - `Playo_DDD_v6_DomainMap.xlsx` → `Game_DDD_v6_DomainMap.xlsx`
   - `Playo_DDD_v7 (1).xlsx` → `Game_DDD_v7 (1).xlsx`
   - `Playo_DDD_v7.xlsx` → `Game_DDD_v7.xlsx`
   - `Playo_DDD_v8.xlsx` → `Game_DDD_v8.xlsx`
3. THE Rename_Tool SHALL rename `Playo_DDD_v8_Diagrams.md` to `Game_DDD_v8_Diagrams.md`.
4. IF a target file name already exists at the destination path, THEN THE Rename_Tool SHALL abort the rename for that file and log a conflict warning, leaving both files intact.
5. WHEN a file is renamed, THE Rename_Tool SHALL update every Cross_Reference to that file found in any Text_File within the workspace.

---

### Requirement 3: Cross-Reference Integrity

**User Story:** As a developer, I want all internal links and references updated after the rename, so that no document points to a file path that no longer exists.

#### Acceptance Criteria

1. AFTER the Rename_Tool completes all file renames, THE Rename_Tool SHALL scan every Text_File for Cross_References that contain the old file names and replace them with the new file names.
2. THE Rename_Tool SHALL update the `file_path` and `workbook_name` fields in `analysis/excel_inventory/inventory.json` to reflect the new Excel file names.
3. THE Rename_Tool SHALL update the `workbook_name` field in `analysis/excel_inventory/data_dictionary.json` if that field contains any Reference.
4. THE Rename_Tool SHALL update the `analysis/excel_inventory/summary.txt` file to replace any Reference in its content.
5. WHEN the Python script `scripts/excel_inventory.py` contains string literals or comments referencing `Playo`, THE Rename_Tool SHALL replace those References with the Canonical_Mapping equivalent.

---

### Requirement 4: Excel Workbook Content Rename

**User Story:** As a developer, I want the cell content inside Excel workbooks updated where "Playo" appears as a product name string, so that the workbooks are internally consistent with the rename.

#### Acceptance Criteria

1. WHEN the Rename_Tool processes an Excel_File, THE Rename_Tool SHALL replace every cell value that is a string containing `Playo` with the equivalent string containing `Game`, applying the Canonical_Mapping.
2. THE Rename_Tool SHALL preserve all cell formatting, formulas, sheet structure, and non-string data types in each Excel_File unchanged.
3. IF an Excel_File is password-protected or cannot be opened by the openpyxl library, THEN THE Rename_Tool SHALL log the file path and skip that file without aborting the overall operation.
4. THE Rename_Tool SHALL update the workbook's sheet names if any sheet name contains `Playo`.
5. AFTER modifying an Excel_File, THE Rename_Tool SHALL save the file to the new renamed path (per Requirement 2) and SHALL NOT retain the old file at the original path.

---

### Requirement 5: Verification and Reporting

**User Story:** As a developer, I want a machine-generated Verification_Report after the rename, so that I can confirm no stale References remain and the operation was complete.

#### Acceptance Criteria

1. AFTER all rename operations complete, THE Rename_Tool SHALL produce a Verification_Report listing the total count of References replaced, grouped by file.
2. THE Rename_Tool SHALL perform a post-rename scan of all Text_Files and report any remaining occurrences of `Playo` or `playo` as unresolved References in the Verification_Report.
3. IF the post-rename scan finds zero remaining References, THEN THE Rename_Tool SHALL mark the Verification_Report status as `COMPLETE`.
4. IF the post-rename scan finds one or more remaining References, THEN THE Rename_Tool SHALL mark the Verification_Report status as `INCOMPLETE` and list each unresolved Reference with its file path and line number.
5. THE Rename_Tool SHALL write the Verification_Report to `analysis/rename_verification.txt` upon completion.

---

### Requirement 6: Idempotency and Safety

**User Story:** As a developer, I want the rename operation to be safe to re-run, so that running it twice does not corrupt files or produce double-substitutions.

#### Acceptance Criteria

1. WHEN the Rename_Tool is executed on a workspace where the rename has already been applied, THE Rename_Tool SHALL detect zero References and produce a Verification_Report with status `COMPLETE` without modifying any file.
2. THE Rename_Tool SHALL NOT replace the string `Game` with `GameGame` or any other doubled form under any execution sequence.
3. WHEN the Rename_Tool encounters a file that has already been renamed to the target name, THE Rename_Tool SHALL skip the rename step for that file and log a skip notice.
4. THE Rename_Tool SHALL process file content replacement before file renaming, so that Cross_Reference updates within files are applied to the correct pre-rename paths.
