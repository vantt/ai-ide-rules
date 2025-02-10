# .cursorrules

name: "ObsidianVideoSummarizer"
version: "2.0.0"
description: >
  This file defines the entire video analysis and summarization process, including:
   1. Downloading subtitles
   2. Analyzing subtitles and extracting key information
   3. Generating a final Vietnamese report based on video_template.md

rules:

  # 1. Subtitles Download
  - name: "Download Subtitles"
    description: >
      Use yt-dlp to retrieve subtitles from YouTube.  Prioritize downloading Vietnamese (zh/zh-CN/zh-Hans),
      and if no Vietnamese subtitles are available, use English (en).
    command: |
      pdm run yt-dlp \
        --proxy 0.0.0.0:1087 \
        --skip-download \
        --write-auto-subs \
        --sub-format vtt \
        --convert-subs vtt \
        --write-subs \
        --sub-lang "vi,vi-VN,vn,en" \
        --output "video_analysis/%(id)s/%(title)s.%(ext)s" \
        $VIDEO_URL

  # 2. Analysis & Information Extraction
  - name: "Analyze Subtitles & Extract Information"
    description: >
      Extract key content from the downloaded subtitle files to generate analysis data.  The goals of this process include:
       - Translate or rewrite English titles into Chinese.
       - Obtain basic video information (duration, publish date, link, etc.).
       - Write a one-sentence summary (at least 50 characters, no line breaks).
       - Summarize the content summary and key points for later use in mind maps and detailed descriptions.
    steps:
      - "Read and clean .vtt/.srt subtitle files, removing timestamps and redundancies"
      - "Obtain the video title, video link, duration, publish date, and other basic information"
      - "If the title is in English, translate or shorten it into Chinese for consistent use."
      - "Based on the subtitle content, initially categorize: main points, key information, practical techniques"
      - "Create a one-sentence introduction (≥50 characters), ensuring no line breaks"
      - |
        When generating mind map nodes:
         1) Include at least a three-level structure: root -> main branch -> sub-branch (can add sub-sub-branches).
         2) Cover the video's introduction/background, chapter divisions, core arguments, important cases or technical points, application scenarios, future developments/summaries, etc.
         3) Ensure each main branch has 2-3 sub-nodes, and sub-nodes should further detail key points based on subtitles or video content.
      - "Generate a detailed description document or draft of at least 1000 words for later template rendering"

  # 3. Generate Final Summary
  - name: "Generate Final Summary"
    description: >
      Based on the extracted content from the previous step, use video_template.md to generate the final report.  Requirements:
       - Title: Chinese
       - One-sentence introduction: ≥50 characters, single line
       - Mind map: at least four-level nodes
       - Video detailed description: ≥1000 words, clear structure, helping readers understand the video in depth
       - Naming: {Y-m-d} {Simplified Chinese video description} - Summary.md
       - Formatting requirements:
         - Do not use H1 headings (the file name is the title)
         - Use a simplified format for video playback: ![{{Title}}]({{URL}})
         - The detailed description section must include:
           * Video background
           * Core arguments
           * Data/cases
           * Application scenarios
           * Future trends
           * Personal reflections
    steps:
      - "Read all extracted data (video title, duration, publish date, link, key information, mind map nodes, etc.)"
      - "Call video_template.md and fill in the information into the corresponding placeholders"
      - "Generate the final document, ensuring the file name follows the format: `{Y-m-d} {Simplified Chinese video description} - Summary.md`"
      - "Ensure the mind map has at least 3 levels of hierarchy, with 2-3 sub-nodes under each main branch"
      - "Save to `video_analysis/{video_id}/` directory"
      - "Archive the summary file to the Obsidian work diary directory: `{{ Obsidian directory }} Video Summary`"
      - "Ensure the document format complies with Obsidian usage standards"