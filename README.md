name: Generate Video

on:
  workflow_dispatch:  # Ручной запуск
  schedule:
    - cron: '0 7 * * *'  # Каждый день в 7:00 UTC

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          pip install moviepy edge-tts pillow requests
          sudo apt-get install -y ffmpeg
      
      - name: Run video generation
        run: python generate_video.py
        env:
          INPUT_TEXT: ${{ secrets.INPUT_TEXT }}
      
      - name: Upload video
        uses: actions/upload-artifact@v4
        with:
          name: generated-video
          path: output.mp4
