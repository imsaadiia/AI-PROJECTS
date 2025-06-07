# AI-PROJECTS
# A smart Python-based assistant that detects mood, suggests study strategies, tracks time with a live clock and stopwatch, and helps maintain productive sessions.
import time
import sqlite3
from datetime import datetime
import pytz
from textblob import TextBlob
import keyboard

# --- Database Setup ---
conn = sqlite3.connect('study_logs.db')
c = conn.cursor()
c.execute('''
    CREATE TABLE IF NOT EXISTS study_sessions (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        mood TEXT,
        suggestion TEXT,
        motivation TEXT,
        timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
    )
''')
conn.commit()

# --- Mood Detection Function ---
def detect_mood(text):
    if not text.strip():
        return "Neutral"
    analysis = TextBlob(text)
    polarity = analysis.sentiment.polarity
    if polarity > 0.3:
        return "Happy"
    elif polarity < -0.3:
        return "Sad"
    else:
        return "Neutral"

# --- Study Suggestion & Motivation ---
def get_suggestion_and_motivation(mood):
    suggestions = {
        "Happy": "Keep your positive energy! Try a new challenging topic for 25 minutes.",
        "Sad": "Don't worry, try a light revision session for 15 minutes to ease in.",
        "Neutral": "Do a 20-minute revision of your previous topic."
    }
    motivations = {
        "Happy": "Your positivity is your strength! Keep pushing forward!",
        "Sad": "Every step counts. You're doing great, keep going!",
        "Neutral": "Stay consistent, even when it's hard."
    }
    return suggestions.get(mood, ""), motivations.get(mood, "")

# --- Logging Function ---
def log_study_session(mood, suggestion, motivation):
    c.execute('INSERT INTO study_sessions (mood, suggestion, motivation) VALUES (?, ?, ?)',
              (mood, suggestion, motivation))
    conn.commit()

# --- Real-Time Clock ---
def show_clock(timezone_str='Asia/Dhaka'):
    try:
        timezone = pytz.timezone(timezone_str)
        now = datetime.now(timezone)
        return now.strftime("%Y-%m-%d %H:%M:%S")
    except Exception as e:
        return f"Invalid timezone: {e}"

# --- Stopwatch with keyboard control ---
def stopwatch():
    print("⏱️ Stopwatch ready! Press 's' to START, and while running press 'e' to STOP.\n")
    print("Press 's' to start the stopwatch...")
    while True:
        if keyboard.is_pressed('s'):
            start_time = time.time()
            print("Stopwatch started! Press 'e' to stop.")
            while True:
                elapsed = time.time() - start_time
                mins = int(elapsed // 60)
                secs = int(elapsed % 60)
                print(f"\r⏱️ Elapsed Time: {mins} minute(s) and {secs} second(s)   ", end="")
                time.sleep(0.5)
                if keyboard.is_pressed('e'):
                    print()
                    print(f"⏹️ Stopwatch stopped at {mins} minute(s) and {secs} second(s).\n")
                    return
        time.sleep(0.1)

# --- Main Program ---
def main():
    print("👋 Welcome to your AI Study Assistant!\n")

    mood_input = input("How are you feeling today? (Press Enter to use default - Neutral): ")
    mood = detect_mood(mood_input or "Neutral")

    suggestion, motivation = get_suggestion_and_motivation(mood)
    print(f"\n🤖 Detected Mood: {mood}")
    print(f"📌 Study Suggestion: {suggestion}")
    print(f"💡 Motivation: {motivation}\n")

    log_study_session(mood, suggestion, motivation)

    print("📍 Select a timezone (e.g., Asia/Dhaka, Europe/London, America/New_York)")
    selected_tz = input("🌐 Enter timezone or press Enter for default (Asia/Dhaka): ") or "Asia/Dhaka"
    print(f"🕒 Current Time in {selected_tz}: {show_clock(selected_tz)}\n")

    use_stopwatch = input("Do you want to use the stopwatch? (yes/no): ").lower()
    if use_stopwatch == 'yes':
        stopwatch()
    else:
        print("⏹️ Stopwatch skipped.\n")

    print("📘 Study session completed. Keep up the great work! 👏")

if __name__ == "__main__":
    main()
