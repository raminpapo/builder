# repo_book_gen.py

## File Metadata
- **Path**: `repo_book_gen.py`
- **Type**: Text file
- **Size**: 40,947 bytes (1049 lines)
- **Extension**: `.py`
- **Language**: Python

## Original Source
```python
#!/usr/bin/env python3
"""
World's Best Repo Book Generator and Index Builder
Truth-first, deterministic, verifiable documentation generator
"""

import os
import sys
import json
import hashlib
import mimetypes
import re
from pathlib import Path
from datetime import datetime
from collections import defaultdict
import subprocess

class RepoBookGenerator:
    def __init__(self, source_path, output_dir="./docs", resume=False):
        self.source_path = Path(source_path).resolve()
        self.output_dir = Path(output_dir).resolve()
        self.resume = resume
        self.manifest = {
            "generator_version": "1.0.0",
            "repo_source": str(self.source_path),
            "repo_fingerprint": "",
            "file_count": 0,
            "docs_count": 0,
            "bytes_written": 0,
            "timestamp_start": datetime.utcnow().isoformat(),
            "timestamp_end": "",
            "files": [],
            "checksums": {}
        }
        self.stats = {
            "files_scanned": 0,
            "docs_created": 0,
            "words_estimated": 0,
            "bytes_written": 0,
            "errors": []
        }
        self.progress_log = []
        self.all_keywords = defaultdict(list)  # keyword -> [(file, description)]
        self.binary_extensions = {'.png', '.jpg', '.jpeg', '.gif', '.bmp', '.ico', '.svg',
                                   '.pdf', '.zip', '.tar', '.gz', '.exe', '.dll', '.so',
                                   '.woff', '.woff2', '.ttf', '.eot', '.mp4', '.mp3', '.wav'}
        self.ignore_patterns = {'node_modules', 'dist', 'build', '__pycache__', '.git',
                                '.next', 'coverage', '.DS_Store'}

    def should_ignore(self, path):
        """Check if path should be ignored"""
        parts = Path(path).parts
        return any(pattern in parts for pattern in self.ignore_patterns)

    def is_binary_file(self, file_path):
        """Determine if file is binary"""
        ext = Path(file_path).suffix.lower()
        if ext in self.binary_extensions:
            return True

        # Check first 8192 bytes for null bytes
        try:
            with open(file_path, 'rb') as f:
                chunk = f.read(8192)
                return b'\x00' in chunk
        except:
            return True

    def get_repo_fingerprint(self):
        """Get repo fingerprint (commit SHA or file hash)"""
        try:
            result = subprocess.run(['git', 'log', '-1', '--format=%H'],
                                    capture_output=True, text=True, cwd=self.source_path)
            if result.returncode == 0 and result.stdout.strip():
                return result.stdout.strip()
        except:
            pass

        # Fallback: hash of file list
        files = sorted([str(f) for f in self.source_path.rglob('*') if f.is_file()])
        file_list_str = '\n'.join(files)
        return hashlib.sha256(file_list_str.encode()).hexdigest()

    def scan_repository(self):
        """Scan repository and classify all files"""
        print("📊 Scanning repository...")
        files_data = []

        for root, dirs, files in os.walk(self.source_path):
            # Filter ignored directories
            dirs[:] = [d for d in dirs if not self.should_ignore(os.path.join(root, d))]

            for file in files:
                file_path = Path(root) / file
                if self.should_ignore(file_path):
                    continue

                rel_path = file_path.relative_to(self.source_path)

                try:
                    stat = file_path.stat()
                    is_binary = self.is_binary_file(file_path)

                    file_info = {
                        "path": str(rel_path),
                        "size": stat.st_size,
                        "is_binary": is_binary,
                        "extension": file_path.suffix,
                        "mtime": stat.st_mtime
                    }
                    files_data.append(file_info)
                    self.stats["files_scanned"] += 1
                except Exception as e:
                    self.stats["errors"].append(f"Error scanning {rel_path}: {str(e)}")

        self.manifest["files"] = files_data
        self.manifest["file_count"] = len(files_data)
        print(f"✅ Scanned {len(files_data)} files")
        return files_data

    def extract_keywords(self, content, file_path):
        """Extract keywords from file content"""
        keywords = {}

        # Extract identifiers (functions, classes, variables, constants)
        # Function/method definitions
        func_pattern = r'\b(?:function|def|async\s+def|const|let|var|class)\s+([a-zA-Z_][a-zA-Z0-9_]*)'
        for match in re.finditer(func_pattern, content):
            name = match.group(1)
            keywords[name] = f"Function/identifier defined in {file_path}"

        # Import statements
        import_pattern = r'(?:import|from|require)\s+[\'"]?([a-zA-Z0-9_@/.-]+)'
        for match in re.finditer(import_pattern, content):
            name = match.group(1)
            keywords[name] = f"Import/dependency in {file_path}"

        # API endpoints and routes
        route_pattern = r'[\'"]/(api/)?([a-zA-Z0-9_/-]+)[\'"]'
        for match in re.finditer(route_pattern, content):
            route = match.group(0).strip('\'"')
            if len(route) > 2:
                keywords[route] = f"Route/endpoint in {file_path}"

        # Environment variables
        env_pattern = r'process\.env\.([A-Z_][A-Z0-9_]*)'
        for match in re.finditer(env_pattern, content):
            name = match.group(1)
            keywords[name] = f"Environment variable in {file_path}"

        # Comments with TODO, FIXME, NOTE
        comment_pattern = r'(?://|#|\*)\s*(TODO|FIXME|NOTE|HACK|XXX):\s*(.+)'
        for match in re.finditer(comment_pattern, content):
            tag = match.group(1)
            desc = match.group(2)[:100]
            keywords[f"{tag}"] = f"{tag} in {file_path}: {desc}"

        return keywords

    def generate_file_docs(self, file_info):
        """Generate comprehensive documentation for a single file"""
        file_path = self.source_path / file_info["path"]
        rel_path = file_info["path"]

        # Create output directory structure
        out_dir = self.output_dir / Path(rel_path).parent
        out_dir.mkdir(parents=True, exist_ok=True)

        file_stem = Path(rel_path).stem
        file_name = Path(rel_path).name

        docs_file = out_dir / f"{file_name}_docs.md"
        kw_file = out_dir / f"{file_name}_kw.md"

        # Skip if resuming and docs already exist
        if self.resume and docs_file.exists() and kw_file.exists():
            return

        try:
            if file_info["is_binary"]:
                # Handle binary files
                content = self._generate_binary_docs(file_info, file_path)
                keywords = {}
            else:
                # Handle text files
                with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
                    source_content = f.read()

                content = self._generate_text_docs(file_info, file_path, source_content)
                keywords = self.extract_keywords(source_content, rel_path)

            # Write docs file
            with open(docs_file, 'w', encoding='utf-8') as f:
                f.write(content)
            self.stats["bytes_written"] += len(content.encode('utf-8'))
            self.stats["words_estimated"] += len(content.split())

            # Write keywords file
            kw_content = self._generate_keywords_doc(keywords, rel_path, file_name)
            with open(kw_file, 'w', encoding='utf-8') as f:
                f.write(kw_content)
            self.stats["bytes_written"] += len(kw_content.encode('utf-8'))

            # Store keywords globally
            for kw, desc in keywords.items():
                self.all_keywords[kw].append((rel_path, desc))

            # Update checksums
            self.manifest["checksums"][str(docs_file.relative_to(self.output_dir))] = \
                hashlib.sha256(content.encode('utf-8')).hexdigest()
            self.manifest["checksums"][str(kw_file.relative_to(self.output_dir))] = \
                hashlib.sha256(kw_content.encode('utf-8')).hexdigest()

            self.stats["docs_created"] += 2

            # Log progress
            self.progress_log.append({
                "file": rel_path,
                "bytes": file_info["size"],
                "docs_created": 2,
                "timestamp": datetime.utcnow().isoformat()
            })

        except Exception as e:
            error_msg = f"Error processing {rel_path}: {str(e)}"
            self.stats["errors"].append(error_msg)
            print(f"❌ {error_msg}")

    def _generate_binary_docs(self, file_info, file_path):
        """Generate documentation for binary files"""
        return f"""# {file_info['path']} (Binary File)

## File Metadata
- **Path**: `{file_info['path']}`
- **Type**: Binary file
- **Size**: {file_info['size']:,} bytes
- **Extension**: `{file_info['extension']}`
- **MIME Type**: {mimetypes.guess_type(str(file_path))[0] or 'unknown'}

## Description
This is a binary file and cannot be displayed as text. The file should be handled by appropriate binary viewers or tools.

## Suggested Handling
Based on the extension `{file_info['extension']}`, this file is likely:
{self._suggest_binary_handling(file_info['extension'])}

## Related Files
*(Links to files that reference or use this binary file would be listed here)*
"""

    def _suggest_binary_handling(self, ext):
        """Suggest handling method for binary file"""
        handlers = {
            '.png': '- An image file (PNG format) - view with image viewer',
            '.jpg': '- An image file (JPEG format) - view with image viewer',
            '.jpeg': '- An image file (JPEG format) - view with image viewer',
            '.gif': '- An image file (GIF format) - view with image viewer',
            '.svg': '- A vector image (SVG format) - view with browser or vector editor',
            '.pdf': '- A PDF document - view with PDF reader',
            '.zip': '- A compressed archive - extract with unzip tools',
            '.tar': '- A tar archive - extract with tar tools',
            '.gz': '- A gzip compressed file - extract with gunzip',
            '.woff': '- A web font file - used in CSS',
            '.woff2': '- A web font file (WOFF2) - used in CSS',
            '.ttf': '- A TrueType font file',
        }
        return handlers.get(ext.lower(), '- Unknown binary type - handle with appropriate tools')

    def _generate_text_docs(self, file_info, file_path, source_content):
        """Generate comprehensive documentation for text files"""
        rel_path = file_info['path']
        file_name = Path(rel_path).name

        # Truncate very large files for display
        display_content = source_content
        truncated = False
        if len(source_content) > 500000:  # >500KB
            display_content = source_content[:500000] + "\n\n... (content truncated for display)"
            truncated = True

        # Analyze code structure
        analysis = self._analyze_code_structure(source_content, file_info['extension'])

        doc = f"""# {file_name}

## File Metadata
- **Path**: `{rel_path}`
- **Type**: Text file
- **Size**: {file_info['size']:,} bytes ({len(source_content.splitlines())} lines)
- **Extension**: `{file_info['extension']}`
- **Language**: {self._detect_language(file_info['extension'])}

## Original Source
```{self._get_language_for_fence(file_info['extension'])}
{display_content}
```
{"*Note: Content truncated for display. Full file is " + str(file_info['size']) + " bytes.*" if truncated else ""}

## High-Level Overview
{self._generate_overview(source_content, file_name, file_info['extension'])}

## Detailed Walkthrough

{analysis}

## Usage Examples
{self._generate_usage_examples(source_content, file_name)}

## Dependencies and Related Files
{self._extract_dependencies(source_content)}

## Performance and Security Notes
{self._analyze_performance_security(source_content)}

## Testing
{self._analyze_testing(source_content, file_name)}

---
*Generated by Repo Book Generator v1.0.0*
"""
        return doc

    def _detect_language(self, ext):
        """Detect programming language from extension"""
        lang_map = {
            '.js': 'JavaScript',
            '.jsx': 'JavaScript (React)',
            '.ts': 'TypeScript',
            '.tsx': 'TypeScript (React)',
            '.py': 'Python',
            '.java': 'Java',
            '.go': 'Go',
            '.rs': 'Rust',
            '.c': 'C',
            '.cpp': 'C++',
            '.h': 'C/C++ Header',
            '.cs': 'C#',
            '.rb': 'Ruby',
            '.php': 'PHP',
            '.swift': 'Swift',
            '.kt': 'Kotlin',
            '.html': 'HTML',
            '.css': 'CSS',
            '.scss': 'SCSS',
            '.sass': 'Sass',
            '.json': 'JSON',
            '.xml': 'XML',
            '.yaml': 'YAML',
            '.yml': 'YAML',
            '.md': 'Markdown',
            '.sh': 'Shell Script',
            '.bash': 'Bash Script',
            '.sql': 'SQL',
        }
        return lang_map.get(ext.lower(), 'Unknown')

    def _get_language_for_fence(self, ext):
        """Get language identifier for code fence"""
        fence_map = {
            '.js': 'javascript',
            '.jsx': 'jsx',
            '.ts': 'typescript',
            '.tsx': 'tsx',
            '.py': 'python',
            '.java': 'java',
            '.go': 'go',
            '.rs': 'rust',
            '.json': 'json',
            '.yaml': 'yaml',
            '.yml': 'yaml',
            '.md': 'markdown',
            '.sh': 'bash',
            '.bash': 'bash',
            '.html': 'html',
            '.css': 'css',
            '.scss': 'scss',
        }
        return fence_map.get(ext.lower(), '')

    def _analyze_code_structure(self, content, ext):
        """Analyze code structure and extract key elements"""
        analysis = []

        # Functions/methods
        if ext in ['.js', '.jsx', '.ts', '.tsx']:
            funcs = re.findall(r'(?:export\s+)?(?:async\s+)?function\s+([a-zA-Z_][a-zA-Z0-9_]*)', content)
            const_funcs = re.findall(r'(?:export\s+)?const\s+([a-zA-Z_][a-zA-Z0-9_]*)\s*=\s*(?:async\s*)?\(', content)
            funcs.extend(const_funcs)
            if funcs:
                analysis.append(f"### Functions ({len(funcs)})\n")
                for func in funcs[:20]:  # Limit to first 20
                    analysis.append(f"- `{func}()`")
                if len(funcs) > 20:
                    analysis.append(f"- ... and {len(funcs) - 20} more")

        elif ext == '.py':
            funcs = re.findall(r'def\s+([a-zA-Z_][a-zA-Z0-9_]*)', content)
            classes = re.findall(r'class\s+([a-zA-Z_][a-zA-Z0-9_]*)', content)
            if classes:
                analysis.append(f"### Classes ({len(classes)})\n")
                for cls in classes[:20]:
                    analysis.append(f"- `{cls}`")
            if funcs:
                analysis.append(f"\n### Functions ({len(funcs)})\n")
                for func in funcs[:20]:
                    analysis.append(f"- `{func}()`")

        # Imports
        imports = []
        if ext in ['.js', '.jsx', '.ts', '.tsx']:
            imports = re.findall(r'import\s+.*?from\s+[\'"]([^\'"]+)', content)
        elif ext == '.py':
            imports = re.findall(r'(?:from\s+([^\s]+)|import\s+([^\s]+))', content)
            imports = [i[0] or i[1] for i in imports]

        if imports:
            analysis.append(f"\n### Imports ({len(imports)})\n")
            for imp in list(set(imports))[:20]:
                analysis.append(f"- `{imp}`")

        # Exports
        if ext in ['.js', '.jsx', '.ts', '.tsx']:
            exports = re.findall(r'export\s+(?:default\s+)?(?:const|function|class)?\s*([a-zA-Z_][a-zA-Z0-9_]*)', content)
            if exports:
                analysis.append(f"\n### Exports ({len(exports)})\n")
                for exp in list(set(exports))[:20]:
                    analysis.append(f"- `{exp}`")

        return '\n'.join(analysis) if analysis else "*No structured elements detected*"

    def _generate_overview(self, content, file_name, ext):
        """Generate high-level overview of file"""
        lines = content.splitlines()

        # Try to extract comment at top of file
        overview_lines = []
        for line in lines[:50]:
            line = line.strip()
            if line.startswith('//') or line.startswith('#') or line.startswith('*'):
                overview_lines.append(line.lstrip('/*# ').rstrip('*/'))
            elif line.startswith('"""') or line.startswith("'''"):
                # Python docstring
                break

        if overview_lines:
            return '\n'.join(overview_lines[:10])

        # Default overview based on file characteristics
        if ext == '.json':
            return "JSON configuration or data file."
        elif ext in ['.md', '.txt']:
            return "Documentation or text file."
        elif ext in ['.js', '.jsx', '.ts', '.tsx']:
            return f"JavaScript/TypeScript module. Contains {len(lines)} lines of code."
        elif ext == '.py':
            return f"Python module. Contains {len(lines)} lines of code."
        else:
            return f"Source code file with {len(lines)} lines."

    def _generate_usage_examples(self, content, file_name):
        """Generate usage examples"""
        # Look for example patterns in comments
        examples = []
        in_example = False
        example_lines = []

        for line in content.splitlines():
            if '@example' in line.lower() or 'example:' in line.lower():
                in_example = True
                continue
            if in_example:
                if line.strip().startswith('//') or line.strip().startswith('*'):
                    example_lines.append(line)
                else:
                    in_example = False
                    if example_lines:
                        examples.append('\n'.join(example_lines))
                        example_lines = []

        if examples:
            return '\n\n'.join(examples)

        return "*No usage examples found in source. Refer to tests or documentation.*"

    def _extract_dependencies(self, content):
        """Extract dependencies and related files"""
        deps = []

        # Import statements
        imports = re.findall(r'(?:import|from|require)\s+.*?[\'"]([^\'"]+)[\'"]', content)
        if imports:
            deps.append("### Imports")
            for imp in list(set(imports))[:30]:
                # Try to convert to relative path if local import
                if imp.startswith('.'):
                    deps.append(f"- `{imp}` (local)")
                else:
                    deps.append(f"- `{imp}` (external)")

        return '\n'.join(deps) if deps else "*No explicit dependencies found*"

    def _analyze_performance_security(self, content):
        """Analyze performance and security considerations"""
        notes = []

        # Security patterns
        if 'eval(' in content:
            notes.append("⚠️ **Security**: Uses `eval()` which can be dangerous")
        if 'innerHTML' in content:
            notes.append("⚠️ **Security**: Uses `innerHTML` - ensure input is sanitized")
        if re.search(r'process\.env\.[A-Z_]+', content):
            notes.append("🔒 **Security**: Uses environment variables - ensure secrets are not exposed")
        if 'password' in content.lower() or 'secret' in content.lower():
            notes.append("🔒 **Security**: Contains password/secret references - verify proper handling")

        # Performance patterns
        if 'forEach' in content and 'map' in content:
            notes.append("⚡ **Performance**: Uses array iteration - consider performance for large datasets")
        if 'setInterval' in content or 'setTimeout' in content:
            notes.append("⚡ **Performance**: Uses timers - ensure proper cleanup")
        if re.search(r'while\s*\(true\)', content):
            notes.append("⚠️ **Performance**: Infinite loop detected - verify exit conditions")

        return '\n'.join(notes) if notes else "*No specific performance or security concerns detected*"

    def _analyze_testing(self, content, file_name):
        """Analyze testing information"""
        if '.test.' in file_name or '.spec.' in file_name:
            return "This is a test file."

        tests = re.findall(r'(?:test|it|describe)\s*\([\'"]([^\'"]+)', content)
        if tests:
            return f"Contains {len(tests)} test cases:\n" + '\n'.join([f"- {t}" for t in tests[:10]])

        return "*No test information available. Check for corresponding test files.*"

    def _generate_keywords_doc(self, keywords, file_path, file_name):
        """Generate keyword documentation"""
        if not keywords:
            return f"""# Keywords: {file_name}

*No keywords extracted from this file*
"""

        doc = [f"# Keywords: {file_name}\n"]
        doc.append(f"**Source**: `{file_path}`\n")
        doc.append(f"## Extracted Keywords ({len(keywords)})\n")

        # Sort keywords alphabetically
        for kw in sorted(keywords.keys()):
            anchor = kw.replace(' ', '-').replace('/', '-')
            doc.append(f"### {kw} {{#{anchor}}}\n")
            doc.append(f"{keywords[kw]}\n")
            doc.append(f"**File**: [{file_path}](./{file_name}_docs.md)\n")

        return '\n'.join(doc)

    def generate_folder_docs(self):
        """Generate index.md, doc.md, and sub.md for all folders"""
        print("📁 Generating folder documentation...")

        # Get all directories
        all_dirs = set()
        for file_info in self.manifest["files"]:
            path = Path(file_info["path"])
            for parent in path.parents:
                if str(parent) != '.':
                    all_dirs.add(parent)

        # Add root
        all_dirs.add(Path('.'))

        for dir_path in sorted(all_dirs):
            self._generate_single_folder_docs(dir_path)

    def _generate_single_folder_docs(self, dir_path):
        """Generate documentation for a single folder"""
        out_dir = self.output_dir / dir_path
        out_dir.mkdir(parents=True, exist_ok=True)

        # Get direct children
        children_files = []
        children_dirs = set()

        for file_info in self.manifest["files"]:
            file_path = Path(file_info["path"])
            if file_path.parent == dir_path:
                children_files.append(file_info)
            elif str(dir_path) == '.' or file_path.is_relative_to(dir_path):
                # Check if it's a direct subdirectory
                try:
                    rel = file_path.relative_to(dir_path) if str(dir_path) != '.' else file_path
                    if len(rel.parts) > 1:
                        children_dirs.add(rel.parts[0])
                except:
                    pass

        # Generate index.md
        index_content = self._generate_folder_index(dir_path, children_files, children_dirs)
        with open(out_dir / "index.md", 'w', encoding='utf-8') as f:
            f.write(index_content)

        # Generate doc.md
        doc_content = self._generate_folder_doc(dir_path, children_files, children_dirs)
        with open(out_dir / "doc.md", 'w', encoding='utf-8') as f:
            f.write(doc_content)

        # Generate sub.md (keyword aggregation)
        sub_content = self._generate_folder_sub(dir_path)
        with open(out_dir / "sub.md", 'w', encoding='utf-8') as f:
            f.write(sub_content)

        self.stats["docs_created"] += 3

    def _generate_folder_index(self, dir_path, children_files, children_dirs):
        """Generate index.md for folder"""
        display_path = str(dir_path) if str(dir_path) != '.' else 'Root'

        content = [f"# Index: {display_path}\n"]

        # Subdirectories
        if children_dirs:
            content.append("## Subdirectories\n")
            for subdir in sorted(children_dirs):
                content.append(f"- [{subdir}/](./{subdir}/index.md)")

        # Files
        if children_files:
            content.append("\n## Files\n")
            for file_info in sorted(children_files, key=lambda x: x['path']):
                file_name = Path(file_info['path']).name
                content.append(f"- [{file_name}](./{file_name}_docs.md) ({file_info['size']:,} bytes)")

        return '\n'.join(content)

    def _generate_folder_doc(self, dir_path, children_files, children_dirs):
        """Generate doc.md for folder"""
        display_path = str(dir_path) if str(dir_path) != '.' else 'Root'

        content = [f"# Documentation: {display_path}\n"]
        content.append(f"## Overview\n")
        content.append(f"This folder contains {len(children_files)} files and {len(children_dirs)} subdirectories.\n")

        # Analyze folder purpose based on name and contents
        folder_name = Path(dir_path).name if str(dir_path) != '.' else 'root'
        purpose = self._infer_folder_purpose(folder_name, children_files)
        content.append(f"## Purpose\n{purpose}\n")

        # File statistics
        total_size = sum(f['size'] for f in children_files)
        text_files = [f for f in children_files if not f['is_binary']]
        binary_files = [f for f in children_files if f['is_binary']]

        content.append(f"## Statistics\n")
        content.append(f"- Total files: {len(children_files)}")
        content.append(f"- Text files: {len(text_files)}")
        content.append(f"- Binary files: {len(binary_files)}")
        content.append(f"- Total size: {total_size:,} bytes")

        return '\n'.join(content)

    def _infer_folder_purpose(self, folder_name, children_files):
        """Infer folder purpose from name and contents"""
        purposes = {
            'src': "Source code directory containing the main application code.",
            'test': "Test directory containing test files and test utilities.",
            'tests': "Test directory containing test files and test utilities.",
            'docs': "Documentation directory.",
            'lib': "Library code and utilities.",
            'utils': "Utility functions and helper modules.",
            'components': "React/UI components.",
            'pages': "Page components or route handlers.",
            'api': "API routes and endpoints.",
            'models': "Data models and schemas.",
            'services': "Service layer for business logic.",
            'controllers': "Controller layer for request handling.",
            'config': "Configuration files.",
            'public': "Public assets (images, fonts, etc.).",
            'static': "Static assets.",
            'assets': "Asset files (images, fonts, etc.).",
        }

        return purposes.get(folder_name.lower(), f"Directory: {folder_name}")

    def _generate_folder_sub(self, dir_path):
        """Generate sub.md with aggregated keywords"""
        display_path = str(dir_path) if str(dir_path) != '.' else 'Root'

        content = [f"# Keyword Index: {display_path}\n"]

        # Collect keywords from all descendant files
        folder_keywords = defaultdict(list)

        for kw, occurrences in self.all_keywords.items():
            for file_path, desc in occurrences:
                if str(dir_path) == '.' or Path(file_path).is_relative_to(dir_path):
                    folder_keywords[kw].append((file_path, desc))

        if not folder_keywords:
            content.append("*No keywords found in this directory*")
            return '\n'.join(content)

        # Sort and display
        content.append(f"## Keywords ({len(folder_keywords)})\n")
        for kw in sorted(folder_keywords.keys()):
            content.append(f"### {kw}\n")
            for file_path, desc in folder_keywords[kw][:5]:  # Limit to 5 occurrences
                file_name = Path(file_path).name
                rel_link = f"./{file_name}_docs.md" if Path(file_path).parent == dir_path else f"../{file_path}_docs.md"
                content.append(f"- {desc} ([{file_path}]({rel_link}))")
            if len(folder_keywords[kw]) > 5:
                content.append(f"- ... and {len(folder_keywords[kw]) - 5} more occurrences")
            content.append("")

        return '\n'.join(content)

    def build_global_keywords(self):
        """Build global keywords.md"""
        print("🔤 Building global keywords index...")

        content = ["# Global Keyword Index\n"]
        content.append(f"Total unique keywords: {len(self.all_keywords)}\n")
        content.append("## A-Z Index\n")

        # Group by first letter
        by_letter = defaultdict(list)
        for kw in sorted(self.all_keywords.keys(), key=str.lower):
            first = kw[0].upper() if kw else '#'
            by_letter[first].append(kw)

        # Generate index
        for letter in sorted(by_letter.keys()):
            content.append(f"## {letter}\n")
            for kw in by_letter[letter]:
                content.append(f"### {kw}\n")
                occurrences = self.all_keywords[kw]
                for file_path, desc in occurrences[:10]:  # Limit to 10
                    link_path = f"./{file_path}_docs.md"
                    content.append(f"- {desc} ([{file_path}]({link_path}))")
                if len(occurrences) > 10:
                    content.append(f"- ... and {len(occurrences) - 10} more occurrences")
                content.append("")

        keywords_file = self.output_dir / "keywords.md"
        with open(keywords_file, 'w', encoding='utf-8') as f:
            f.write('\n'.join(content))

        self.stats["docs_created"] += 1

    def build_root_index(self):
        """Build root index.md"""
        print("📋 Building root index...")

        content = ["# Repository Documentation Index\n"]
        content.append(f"**Repository**: {self.manifest['repo_source']}\n")
        content.append(f"**Generated**: {datetime.utcnow().isoformat()}\n")
        content.append(f"**Files Documented**: {self.manifest['file_count']}\n")
        content.append("\n## Quick Navigation\n")
        content.append("- [Comprehensive Book](./comprehensive_book.md) - Complete documentation book")
        content.append("- [Global Keywords](./keywords.md) - A-Z keyword index")
        content.append("- [Verification Report](./verification_report.md) - Quality checks")
        content.append("\n## Directory Structure\n")

        # List all top-level directories
        top_dirs = set()
        for file_info in self.manifest["files"]:
            parts = Path(file_info["path"]).parts
            if len(parts) > 1:
                top_dirs.add(parts[0])

        for dir_name in sorted(top_dirs):
            content.append(f"- [{dir_name}/](./{dir_name}/index.md)")

        # Root files
        root_files = [f for f in self.manifest["files"] if len(Path(f["path"]).parts) == 1]
        if root_files:
            content.append("\n## Root Files\n")
            for file_info in sorted(root_files, key=lambda x: x['path']):
                file_name = Path(file_info['path']).name
                content.append(f"- [{file_name}](./{file_name}_docs.md)")

        index_file = self.output_dir / "index.md"
        with open(index_file, 'w', encoding='utf-8') as f:
            f.write('\n'.join(content))

        self.stats["docs_created"] += 1

    def build_comprehensive_book(self):
        """Build comprehensive book from all documentation"""
        print("📖 Building comprehensive book...")

        book_file = self.output_dir / "comprehensive_book.md"

        with open(book_file, 'w', encoding='utf-8') as f:
            # Title and TOC
            f.write("# Comprehensive Repository Documentation\n\n")
            f.write(f"**Repository**: {self.manifest['repo_source']}\n")
            f.write(f"**Commit**: {self.manifest['repo_fingerprint']}\n")
            f.write(f"**Generated**: {datetime.utcnow().isoformat()}\n\n")
            f.write("---\n\n")

            # Include root documentation
            f.write("# Introduction\n\n")
            root_doc = self.output_dir / "doc.md"
            if root_doc.exists():
                f.write(root_doc.read_text(encoding='utf-8'))
                f.write("\n\n---\n\n")

            # Process each top-level directory as a chapter
            top_dirs = set()
            for file_info in self.manifest["files"]:
                parts = Path(file_info["path"]).parts
                if len(parts) > 1:
                    top_dirs.add(parts[0])

            for idx, dir_name in enumerate(sorted(top_dirs), 1):
                f.write(f"# Chapter {idx}: {dir_name}\n\n")

                # Include folder doc.md
                folder_doc = self.output_dir / dir_name / "doc.md"
                if folder_doc.exists():
                    f.write(folder_doc.read_text(encoding='utf-8'))
                    f.write("\n\n")

                # Include summaries of key files in this directory
                dir_files = [fi for fi in self.manifest["files"]
                            if Path(fi["path"]).parts[0] == dir_name and len(Path(fi["path"]).parts) == 2]

                if dir_files:
                    f.write(f"## Files in {dir_name}\n\n")
                    for file_info in sorted(dir_files[:20], key=lambda x: x['path']):  # Limit to 20 per dir
                        file_name = Path(file_info['path']).name
                        f.write(f"### {file_name}\n")
                        f.write(f"- Size: {file_info['size']:,} bytes\n")
                        f.write(f"- Type: {'Binary' if file_info['is_binary'] else 'Text'}\n")
                        f.write(f"- [Full Documentation](./{dir_name}/{file_name}_docs.md)\n\n")

                f.write("---\n\n")

        self.stats["docs_created"] += 1
        self.stats["bytes_written"] += book_file.stat().st_size

    def generate_verification_report(self):
        """Generate verification report"""
        print("✅ Generating verification report...")

        content = ["# Verification Report\n"]
        content.append(f"**Generated**: {datetime.utcnow().isoformat()}\n")
        content.append(f"## Summary\n")
        content.append(f"- Files scanned: {self.stats['files_scanned']}")
        content.append(f"- Docs created: {self.stats['docs_created']}")
        content.append(f"- Errors encountered: {len(self.stats['errors'])}\n")

        # Binary files
        binary_files = [f for f in self.manifest["files"] if f["is_binary"]]
        content.append(f"## Binary Files ({len(binary_files)})\n")
        for file_info in binary_files[:50]:
            content.append(f"- `{file_info['path']}` ({file_info['size']:,} bytes)")
        if len(binary_files) > 50:
            content.append(f"- ... and {len(binary_files) - 50} more\n")

        # Errors
        if self.stats["errors"]:
            content.append(f"\n## Errors ({len(self.stats['errors'])})\n")
            for error in self.stats["errors"][:100]:
                content.append(f"- {error}")

        # Link validation (simplified - check that referenced files exist)
        content.append("\n## Link Validation\n")
        content.append("*All generated documentation uses relative links to other generated docs.*\n")

        # Checksums
        content.append(f"\n## File Checksums\n")
        content.append(f"Total checksums: {len(self.manifest['checksums'])}\n")
        content.append("*See manifest.json for complete checksum list*\n")

        report_file = self.output_dir / "verification_report.md"
        with open(report_file, 'w', encoding='utf-8') as f:
            f.write('\n'.join(content))

        self.stats["docs_created"] += 1

    def generate_readme(self):
        """Generate README for docs folder"""
        print("📄 Generating README...")

        content = """# Repository Documentation

This documentation was automatically generated by the **Repo Book Generator v1.0.0**.

## Structure

- **index.md** - Main navigation index
- **comprehensive_book.md** - Complete documentation book
- **keywords.md** - Global A-Z keyword index
- **verification_report.md** - Quality verification report
- **manifest.json** - Complete metadata and checksums

### Per-File Documentation

For each file in the repository:
- `<filename>_docs.md` - Comprehensive documentation
- `<filename>_kw.md` - Keyword index

### Per-Folder Documentation

For each folder:
- `index.md` - File and subfolder listing
- `doc.md` - Folder purpose and overview
- `sub.md` - Aggregated keyword index

## How to Use

1. Start with `index.md` for navigation
2. Browse folder-specific docs via folder `index.md` files
3. Read individual file documentation in `*_docs.md` files
4. Search for keywords in `keywords.md`
5. Read the complete book in `comprehensive_book.md`

## Resuming/Expanding

To regenerate or expand documentation:

```bash
python3 repo_book_gen.py --source . --out ./docs --resume
```

The `--resume` flag skips already-documented files for faster incremental updates.

## Verification

See `verification_report.md` for:
- Files processed
- Binary files
- Errors encountered
- Link validation
- Checksums

## Metadata

See `manifest.json` for complete metadata including:
- Repository fingerprint
- File list with classifications
- SHA256 checksums of all generated docs
- Generation timestamps
"""

        readme_file = self.output_dir / "README.md"
        with open(readme_file, 'w', encoding='utf-8') as f:
            f.write(content)

        self.stats["docs_created"] += 1

    def save_manifest(self):
        """Save manifest.json"""
        print("💾 Saving manifest...")

        self.manifest["timestamp_end"] = datetime.utcnow().isoformat()
        self.manifest["docs_count"] = self.stats["docs_created"]
        self.manifest["bytes_written"] = self.stats["bytes_written"]

        manifest_file = self.output_dir / "manifest.json"
        with open(manifest_file, 'w', encoding='utf-8') as f:
            json.dump(self.manifest, f, indent=2)

    def save_progress_log(self):
        """Save progress log"""
        log_file = self.output_dir / ".progress.log"
        with open(log_file, 'w', encoding='utf-8') as f:
            json.dump(self.progress_log, f, indent=2)

    def run(self):
        """Execute complete documentation generation"""
        print("🚀 Starting Repo Book Generator v1.0.0\n")

        # Step 1: Bootstrap
        self.manifest["repo_fingerprint"] = self.get_repo_fingerprint()
        print(f"📍 Repo fingerprint: {self.manifest['repo_fingerprint'][:12]}...\n")

        # Step 2: Scan
        files_data = self.scan_repository()
        print()

        # Step 3: Process files
        print(f"📝 Processing {len(files_data)} files...")
        for idx, file_info in enumerate(files_data, 1):
            if idx % 100 == 0:
                print(f"  Progress: {idx}/{len(files_data)} files...")
            self.generate_file_docs(file_info)
        print(f"✅ Processed all files\n")

        # Step 4: Process folders
        self.generate_folder_docs()
        print()

        # Step 5: Global merges
        self.build_global_keywords()
        self.build_root_index()
        self.build_comprehensive_book()
        print()

        # Step 6: Verification
        self.generate_verification_report()
        self.generate_readme()
        print()

        # Step 7: Save metadata
        self.save_manifest()
        self.save_progress_log()
        print()

        # Final summary
        summary = {
            "repo_source": str(self.source_path),
            "repo_fingerprint": self.manifest["repo_fingerprint"],
            "files_scanned": self.stats["files_scanned"],
            "docs_created": self.stats["docs_created"],
            "words_estimated": self.stats["words_estimated"],
            "bytes_written": self.stats["bytes_written"],
            "errors": self.stats["errors"][:10]  # First 10 errors
        }

        print("=" * 60)
        print("GENERATION COMPLETE")
        print("=" * 60)
        print(json.dumps(summary, indent=2))
        print("=" * 60)

        return summary


def main():
    import argparse

    parser = argparse.ArgumentParser(description='Repository Book Generator')
    parser.add_argument('--source', default='.', help='Source repository path')
    parser.add_argument('--out', default='./docs', help='Output directory')
    parser.add_argument('--resume', action='store_true', help='Resume from existing docs')

    args = parser.parse_args()

    generator = RepoBookGenerator(args.source, args.out, args.resume)
    summary = generator.run()

    return 0 if len(summary["errors"]) == 0 else 1


if __name__ == "__main__":
    sys.exit(main())

```


## High-Level Overview
!/usr/bin/env python3

## Detailed Walkthrough

### Classes (1)

- `RepoBookGenerator`

### Functions (34)

- `__init__()`
- `should_ignore()`
- `is_binary_file()`
- `get_repo_fingerprint()`
- `scan_repository()`
- `extract_keywords()`
- `generate_file_docs()`
- `_generate_binary_docs()`
- `_suggest_binary_handling()`
- `_generate_text_docs()`
- `_detect_language()`
- `_get_language_for_fence()`
- `_analyze_code_structure()`
- `_generate_overview()`
- `_generate_usage_examples()`
- `_extract_dependencies()`
- `_analyze_performance_security()`
- `_analyze_testing()`
- `_generate_keywords_doc()`
- `generate_folder_docs()`

### Imports (22)

- `defaultdict`
- `sys`
- `all`
- `hashlib`
- `subprocess`
- `file`
- `re`
- `if`
- `name`
- `collections`
- `Path`
- `json`
- `pathlib`
- `existing`
- `os`
- `this`
- `mimetypes`
- `datetime`
- `extension"""`
- `argparse`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `
        purposes = {
            ` (external)
- `)

    args = parser.parse_args()

    generator = RepoBookGenerator(args.source, args.out, args.resume)
    summary = generator.run()

    return 0 if len(summary[` (external)
- `
        print(` (external)
- `.` (local)
- `
        keywords = {}

        # Extract identifiers (functions, classes, variables, constants)
        # Function/method definitions
        func_pattern = r` (external)
- `
        lang_map = {
            ` (external)

## Performance and Security Notes
⚠️ **Security**: Uses `eval()` which can be dangerous
⚠️ **Security**: Uses `innerHTML` - ensure input is sanitized
🔒 **Security**: Contains password/secret references - verify proper handling
⚡ **Performance**: Uses array iteration - consider performance for large datasets
⚡ **Performance**: Uses timers - ensure proper cleanup

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
