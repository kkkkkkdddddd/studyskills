---
name: testpraper
description: An intelligent tool for generating styled academic review/exam manuals (PDFs) from PowerPoint (PPTX) presentations or raw texts. It completely transforms raw PPT text into a highly structured, professional exam manual format mimicking the "complex_network_zero_to_exam_manual" style, complete with colored LaTeX boxes (blue for concepts, orange for steps) and proper sectioning. Trigger this when the user mentions testpraper, or asks to parse PPTs to generate a styled exam manual or summary PDF.
---

# testpraper: Course Manual PDF Generator

This skill processes PPTX files or raw text into a professional, LaTeX-styled PDF review manual designed for exam preparation.

## Execution Workflow

When the user triggers this skill, follow these specific steps:

### Step 1: Text Extraction
If the input is a set of `.pptx` files, write a script to extract text from all slides in the PPTX files.
*(Hint: Since `python-pptx` might not be installed, you can use `zipfile` and `xml.etree.ElementTree` to parse the `ppt/slides/slide*.xml` files inside the `.pptx` zip.)*
Combine all extracted texts into a raw text file `extracted_all.txt`.

### Step 2: Content Synthesis & Structuring
Read the extracted text. Do not mechanically copy it. Act as a professional tutor writing a "zero-to-exam sprint manual" (零基础冲刺讲义).
- Categorize concepts into structured sections (`\section`, `\subsection`).
- Identify **Key Insights, Theories, and Conclusions** and wrap them in Blue Boxes (`\begin{bluebox}[标题] ... \end{bluebox}`).
- Identify **Processes, Workflows, Exam Questions, and Steps** and wrap them in Orange Boxes (`\begin{orangebox}[标题] ... \end{orangebox}`).
- Identify **Example Questions and Applications** (especially near formulas) and wrap them in Green Boxes (`\begin{examplebox}[例题：...] ... \end{examplebox}`)。
- **Selective Variable Explanation**: 当公式或总结中出现非基础、可能不易理解的数学变量或统计学指标时（例如 $SS_x$, $\sigma^2$, $\tau$ 等），必须在首次出现时专门解释其“物理意义”与“数学构成”（例如：$SS_x$ 代表自变量 $x$ 的离差平方和 $\sum(x_i - \bar{x})^2$，反映了数据在分布上的跨度大小）。注意：只需针对复杂核心变量解释，不要对过于基础的简单字母（如通用的 $x, y$）过度解释，保持讲义的专业和精炼。
- **vivid example：**在遇到一些抽象的核心内容时，举一个具体的例子（算例）辅助理解


### Step 3: Construction using the Target Template

Generate a LaTeX file embedding the synthesized content within this strict LaTeX preamble structure:

```latex
\documentclass[11pt,a4paper]{article}
\usepackage[UTF8]{ctex}
\usepackage[dvipsnames]{xcolor}
\usepackage[most]{tcolorbox}
\usepackage{geometry}
\geometry{a4paper, margin=2.5cm}
\usepackage{titlesec}
\usepackage[colorlinks=true, linkcolor=blue, urlcolor=blue]{hyperref}
\usepackage{graphicx}
\usepackage{tikz}

% Colors
\definecolor{conclusionBlue}{RGB}{235, 245, 255}
\definecolor{conclusionBorder}{RGB}{41, 128, 185}
\definecolor{stepOrange}{RGB}{255, 245, 235}
\definecolor{stepBorder}{RGB}{230, 126, 34}
\definecolor{exampleGreen}{RGB}{235, 255, 240}
\definecolor{exampleBorder}{RGB}{46, 204, 113}

% Blue box (Concepts, Conclusions)
\newtcolorbox{bluebox}[1][]{
    enhanced, breakable, colback=conclusionBlue, colframe=conclusionBorder,
    fonttitle=\bfseries\sffamily, title=#1, boxrule=1.2pt, arc=4pt,
    left=10pt, right=10pt, top=8pt, bottom=8pt
}

% Orange box (Steps, Workflows, Questions)
\newtcolorbox{orangebox}[1][]{
    enhanced, breakable, colback=stepOrange, colframe=stepBorder,
    fonttitle=\bfseries\sffamily, title=#1, boxrule=1.2pt, arc=4pt,
    left=10pt, right=10pt, top=8pt, bottom=8pt
}

% Green box (Examples, Applications)
\newtcolorbox{examplebox}[1][]{
    enhanced, breakable, colback=exampleGreen, colframe=exampleBorder,
    fonttitle=\bfseries\sffamily, title=#1, boxrule=1.2pt, arc=4pt,
    left=10pt, right=10pt, top=8pt, bottom=8pt
}

\titleformat{\section}{\Large\bfseries\sffamily}{\thesection}{1em}{}
\titleformat{\subsection}{\large\bfseries\sffamily}{\thesubsection}{1em}{}

\title{\vspace{-1.5cm}\Huge{\textbf{总复习冲刺讲义}}\\\vspace{0.8cm}\large{从头梳理全套知识体系}}
\author{}
\date{}

\begin{document}
\maketitle
\centerline{\rule{0.8\textwidth}{0.5pt}}
\vspace{0.5cm}
\noindent\textbf{\Large{最推荐的使用方式}}\\
第一遍，只读\textbf{\textcolor{conclusionBorder}{蓝框“结论与直觉”}}和\textbf{\textcolor{stepBorder}{橙框“核心流程”}}；第二遍，结合\textbf{\textcolor{exampleBorder}{绿框“例题”}}把详细机制读懂。
\vspace{1cm}
\tableofcontents
\newpage

% --- YOUR SYNTHESIZED CONTENT HERE ---
% Use \section{}, \subsection{}
% Use \begin{bluebox}[...] , \begin{orangebox}[...] and \begin{examplebox}[...]

\end{document}
```

### Step 4: Write and Compile
1. Use `write_to_file` to output the final `.tex` file.
2. Use `run_command` to execute `xelatex -interaction=nonstopmode filename.tex` twice (to properly compile the table of contents).
3. Notify the user the PDF is ready.
