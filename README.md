# MsBuildDump

A CLI tool that parses MSBuild solutions (.sln) and projects (.vcxproj, .csproj, etc.) and outputs structured JSON for use with RAG/code indexing tools.

## Prerequisites

- Windows 10/11
- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- Visual Studio 2022 (or Build Tools) installed — required for MSBuild to locate compilers

## Setup

### Option 1: Command Line

```powershell
# Clone or download this folder, then:
cd msbuild-dump
dotnet restore
dotnet build
```

### Option 2: Visual Studio

1. Open `MsBuildDump.sln` in Visual Studio 2022
2. Build the solution (Ctrl+Shift+B)

## Usage

### Basic Usage

```powershell
# Run from project directory
dotnet run --project MsBuildDump -- "C:\path\to\YourSolution.sln"

# Output to file
dotnet run --project MsBuildDump -- "C:\path\to\YourSolution.sln" output.json
```

### After Building

```powershell
# From bin directory
.\MsBuildDump.exe "C:\path\to\YourSolution.sln"

# Output to file
.\MsBuildDump.exe "C:\path\to\YourSolution.sln" output.json
```

### Publish as Single Executable

```powershell
dotnet publish -c Release

# Find the exe at:
# MsBuildDump\bin\Release\net8.0\win-x64\publish\MsBuildDump.exe
```

## Output Format

```json
{
  "Solution": "C:\\path\\to\\YourSolution.sln",
  "SolutionName": "YourSolution",
  "Projects": [
    {
      "Name": "MyProject",
      "Path": "C:\\path\\to\\MyProject\\MyProject.vcxproj",
      "ProjectType": "C++",
      "TargetFramework": null,
      "Configurations": ["Debug", "Release", "x64", "x86"],
      "SourceFiles": [
        "main.cpp",
        "utils.cpp",
        "utils.h"
      ],
      "References": ["CoreLib", "NetworkUtils"],
      "PackageReferences": null,
      "AssemblyReferences": null,
      "IncludePaths": ["src\\common", "third_party\\boost"],
      "Defines": ["WIN32", "_DEBUG", "UNICODE"]
    }
  ],
  "DependencyGraph": {
    "MyProject": ["CoreLib", "NetworkUtils"],
    "CoreLib": []
  }
}
```

## Supported Project Types

| Extension | Type |
|-----------|------|
| `.vcxproj` | C++ |
| `.csproj` | C# |
| `.vbproj` | VB.NET |
| `.fsproj` | F# |
| `.vcproj` | C++ (Legacy VS2008 and earlier) |

## Using Output with Python RAG Tools

```python
import json

with open('output.json', 'r') as f:
    solution = json.load(f)

# Iterate projects
for project in solution['Projects']:
    print(f"Project: {project['Name']} ({project['ProjectType']})")
    for source_file in project['SourceFiles']:
        print(f"  - {source_file}")

# Get dependency order (topological sort)
dep_graph = solution['DependencyGraph']
# ... your indexing logic
```

## Troubleshooting

### "MSBuild not found" or similar errors

Make sure Visual Studio 2022 (or Build Tools for VS 2022) is installed. The tool uses `MSBuildLocator` to find your VS installation.

### Project fails to parse

Some older or unusual project formats may not parse correctly. Check the warnings in stderr — the tool will skip problematic projects and continue.

### Missing source files in output

The tool reads what MSBuild knows about. If files are added via wildcards or complex build logic, they may not all appear.

## Next Steps

This JSON output can feed into:

1. **Tree-sitter parsing** — Use the source file list to parse each file
2. **Vector embedding** — Chunk and embed code for RAG
3. **Code graph construction** — Build symbol/call graphs from the dependency info
4. **Documentation generation** — Generate docs per-project or per-module

## License

MIT — do whatever you want with it.
