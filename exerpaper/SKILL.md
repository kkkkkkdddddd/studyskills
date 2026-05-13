---
name: exerpaper
description: An intelligent tool for generating styled academic exercise solutions and knowledge summaries (PDFs) from PDF/PPTX problem sets. It solves problems step-by-step and formats them into a highly structured, professional solution manual style with knowledge point summaries for each question and a global summary at the end. Trigger this when the user mentions exerpaper, or asks to generate detailed solutions with summaries for a set of problems.
---

# exerpaper: Academic Exercise Solutions & Summary Generator

This skill processes problem set files (like PDFs) into a professional, LaTeX-styled PDF solution manual. It provides detailed mathematical solutions, extracts key knowledge points per question, and concludes with a global topic summary.

## Layout Principle

**Each sub-question's original text appears in a green box immediately above its solution**, rather than grouping all sub-questions together at the front of the main question section. This makes it easy to read the original problem and its solution side by side without scrolling back.

Structure for each main question:
```
\section*{Question X: Title}
[Brief background / data summary]
  └─ \subsection*{(a) ...}
       └─ green qbox with original (a) text
       └─ detailed solution for (a)
  └─ \subsection*{(b) ...}
       └─ green qbox with original (b) text
       └─ detailed solution for (b)
  └─ ...
  └─ blue knowledgebox with QX 知识点总结
```

## Execution Workflow

When the user triggers this skill, follow these specific steps:

### Step 1: Problem Extraction
1. **First, use the `/pdf-converter` skill** to convert the PDF/PPTX exercise file into a Markdown (`.md`) file.
2. **Then read the extracted `.md` file** to understand the full problem set before solving.

### Step 2: Problem Solving & Synthesis
Read the extracted problems. Act as a professional tutor writing a detailed solution manual.
- **Detailed Solutions**: Solve each sub-problem step-by-step, showing all calculations, formulas, and logical reasoning. Use standard LaTeX math formatting.
- **Knowledge Point Summary**: After each main question's all sub-solutions, provide a summary of the concepts and techniques used. Wrap in a Knowledge Box (`\begin{knowledgebox}[Qx 知识点总结] ... \end{knowledgebox}`).
- **Global Summary**: At the end of the document, wrap a comprehensive summary in a Summary Box (`\begin{summarybox}[总总结 (Global Summary)] ... \end{summarybox}`).

### Step 3: Construction using the Target Template
Generate a LaTeX file embedding the synthesized content within this strict LaTeX preamble structure:

```latex
\documentclass[11pt,a4paper]{article}
\usepackage[UTF8]{ctex}
\usepackage{amsmath, amssymb}
\usepackage[dvipsnames]{xcolor}
\usepackage[most]{tcolorbox}
\usepackage{geometry}
\geometry{a4paper, margin=2.5cm}
\usepackage{titlesec}
\usepackage[colorlinks=true, linkcolor=blue, urlcolor=blue]{hyperref}
\usepackage{booktabs}
\usepackage{enumitem}

% Colors
\definecolor{knowledgeBlue}{RGB}{235, 245, 255}
\definecolor{knowledgeBorder}{RGB}{41, 128, 185}
\definecolor{summaryOrange}{RGB}{255, 245, 235}
\definecolor{summaryBorder}{RGB}{230, 126, 34}
\definecolor{questionGreen}{RGB}{235, 255, 240}
\definecolor{questionBorder}{RGB}{39, 174, 96}

% Question Box — for sub-question original text
\newtcolorbox{qbox}[1][]{
    enhanced, colback=questionGreen, colframe=questionBorder,
    fonttitle=\bfseries\sffamily, title=#1, boxrule=0.8pt, arc=3pt,
    left=8pt, right=8pt, top=5pt, bottom=5pt, boxsep=2pt
}

% Knowledge Box
\newtcolorbox{knowledgebox}[1][]{
    enhanced, colback=knowledgeBlue, colframe=knowledgeBorder,
    fonttitle=\bfseries\sffamily, title=#1, boxrule=1.2pt, arc=4pt,
    left=10pt, right=10pt, top=8pt, bottom=8pt
}

% Summary Box
\newtcolorbox{summarybox}[1][]{
    enhanced, colback=summaryOrange, colframe=summaryBorder,
    fonttitle=\bfseries\sffamily, title=#1, boxrule=1.2pt, arc=4pt,
    left=10pt, right=10pt, top=8pt, bottom=8pt
}

\titleformat{\section}{\Large\bfseries\sffamily}{\thesection}{1em}{}
\titleformat{\subsection}{\large\bfseries\sffamily}{\thesubsection}{1em}{}

\title{\vspace{-1.5cm}\Huge{\textbf{习题详细解答与总结}}\\\vspace{0.8cm}\large{Problem Solutions \& Knowledge Summaries}}
\author{}
\date{}

\begin{document}
\maketitle
\centerline{\rule{0.8\textwidth}{0.5pt}}
\vspace{0.5cm}

% --- CONTENT STRUCTURE ---
% \section*{Question X: Title}
% Brief background / data summary
%
% \subsection*{(a) Sub-title [marks]}
% \begin{qbox}[(a) Original Question]
% Original English text of sub-question (a)
% \end{qbox}
% Detailed solution in Chinese with English terminology
%
% \subsection*{(b) Sub-title [marks]}
% \begin{qbox}[(b) Original Question]
% Original English text of sub-question (b)
% \end{qbox}
% Detailed solution...
%
% \begin{knowledgebox}[QX 知识点总结]
% Key concepts and techniques summary
% \end{knowledgebox}
%
% ...repeat for all questions...
%
% \begin{summarybox}[总总结 (Global Summary)]
% Comprehensive cross-question summary
% \end{summarybox}

\end{document}
```

### Step 4: Write and Compile
1. Use `write_to_file` to output the final `.tex` file to the same directory as the input.
2. Compile with: `xelatex -interaction=nonstopmode filename.tex` (run twice for references).
3. If the PDF is locked by a viewer, use `-jobname=filename_v2` as fallback.
4. **Cleanup**: Delete all auxiliary files (`.aux`, `.log`, `.out`, `.tex`, `.toc`, etc.), keeping only the `.md` and `.pdf` files.
5. Notify the user the PDF is ready with the file path.
