import asyncio
import edge_tts
from moviepy.editor import *
from PIL import Image, ImageDraw, ImageFont
import requests
import os

# 1. Разбиваем текст на сцены (по предложениям)
def split_text(text, max_len=100):
    sentences = text.replace('\n', ' ').split('. ')
    scenes = []
    current = ""
    for s in sentences:
        if len(current) + len(s) < max_len:
            current += s + ". "
        else:
            if current:
                scenes.append(current.strip())
            current = s + ". "
    if current:
        scenes.append(current.strip())
    return scenes

# 2. Генерируем озвучку через Edge-TTS (бесплатно)
async def generate_audio(text, output_file, voice="ru-RU-DmitryNeural"):
    communicate = edge_tts.Communicate(text, voice)
    await communicate.save(output_file)

# 3. Создаём простое изображение с текстом (если нет API для картинок)
def create_text_image(text, output_file, size=(1920, 1080)):
    img = Image.new('RGB', size, color=(30, 30, 40))
    draw = ImageDraw.Draw(img)
    try:
        font = ImageFont.truetype("arial.ttf", 60)
    except:
        font = ImageFont.load_default()
    
    # Простой перенос строк
    lines = []
    words = text.split()
    line = ""
    for w in words:
        if len(line + w) < 40:
            line += w + " "
        else:
            lines.append(line)
            line = w + " "
    lines.append(line)
    
    y = 200
    for line in lines:
        draw.text((100, y), line, font=font, fill=(255, 255, 255))
        y += 80
    
    img.save(output_file)

# 4. Собираем видео
def build_video(scenes, output_file="output.mp4"):
    clips = []
    for i, scene in enumerate(scenes):
        audio_file = f"temp_audio_{i}.mp3"
        img_file = f"temp_img_{i}.png"
        
        # Генерируем аудио
        asyncio.run(generate_audio(scene, audio_file))
        # Генерируем изображение
        create_text_image(scene, img_file)
        
        # Загружаем аудио и картинку
        audio = AudioFileClip(audio_file)
        img_clip = ImageClip(img_file).set_duration(audio.duration)
        img_clip = img_clip.set_audio(audio)
        clips.append(img_clip)
        
        # Чистим временные файлы
        os.remove(audio_file)
        os.remove(img_file)
    
    # Конкатенируем все сцены
    final = concatenate_videoclips(clips)
    final.write_videofile(output_file, fps=24, codec='libx264', audio_codec='aac')

if __name__ == "__main__":
    text = input("Введите текст для видео: ")
    scenes = split_text(text)
    print(f"Создано {len(scenes)} сцен. Генерация...")
    build_video(scenes)
    print("Готово! Видео сохранено как output.mp4")# -
Позитивный взгляд на историю Луганска
