import os
import json
import random
import time
import tkinter as tk
from tkinter import messagebox, ttk
import winsound

# ==============================================================================
# 🖥️ HIGH DPI SHARPNESS ENHANCEMENT (Windows Only)
# ==============================================================================
try:
    import ctypes
    # Tells Windows to render the GUI natively at the monitor's exact resolution scaling
    ctypes.windll.shcore.SetProcessDpiAwareness(1)
except Exception:
    # Safe fallback if system permissions or platform differs
    pass

# ==============================================================================
# 🔊 SOUND ENGINE (Windows winsound - Synth Retro Style)
# ==============================================================================
class AudioEngine:
    @staticmethod
    def play_correct():
        winsound.Beep(900, 80)
        winsound.Beep(1300, 100)

    @staticmethod
    def play_wrong():
        winsound.Beep(180, 350)

    @staticmethod
    def play_hint():
        winsound.Beep(800, 600)

    @staticmethod
    def play_win_round():
        frequencies = [440, 554, 659, 880, 1109, 1318]
        for freq in frequencies:
            winsound.Beep(freq, 60)

    @staticmethod
    def play_game_over():
        frequencies = [600, 450, 300, 150]
        for freq in frequencies:
            winsound.Beep(freq, 200)

    @staticmethod
    def play_new_highscore():
        for _ in range(2):
            for freq in [523, 659, 784, 1046]:
                winsound.Beep(freq, 70)

# ==============================================================================
# 🗃️ WORD BANK SYSTEM
# ==============================================================================
WORDS_DICTIONARY = {
    "animals": ["tiger", "zebra", "koala", "sheep", "horse", "mouse", "panda", "snake", "camel", "fox"],
    "fruits": ["apple", "mango", "grape", "peach", "guava", "berry", "lemon", "melon", "plum", "banana"],
    "vegetables": ["carrot", "onion", "tomato", "potato", "radish", "pepper", "celery", "beet", "garlic", "corn"],
    "stationery": ["pen", "pencil", "ruler", "eraser", "marker", "crayon", "notebook", "paper", "glue", "tape"],
    "tech": ["laptop", "tablet", "router", "modem", "mouse", "screen", "server", "mobile", "cable", "python"],
    "sports": ["tennis", "soccer", "boxing", "golf", "rugby", "hockey", "judo", "karate", "cricket", "racing"]
}

DIFFICULTY_CONFIG = {
    "easy": {"lives": 8, "multiplier": 10},
    "normal": {"lives": 6, "multiplier": 15},
    "hard": {"lives": 4, "multiplier": 20}
}

DATA_FILE = "hangman_data.json"

# ==============================================================================
# 🎮 MAIN GUI APPLICATION CLASS
# ==============================================================================
class HangmanGUIApp:
    def __init__(self, root):
        self.root = root
        self.root.title("CONSOLE HANGMAN PRO // RETRO EDITION")
        self.root.geometry("850x650")
        self.root.configure(bg="#000000")  # Absolute Black
        self.root.resizable(False, False)

        # Style Configuration
        self.style = ttk.Style()
        self.style.theme_use("clam")

        # Load Statistics Data
        self.system_profile_data = self.load_game_data()

        # Initialize Transient Game States
        self.current_round = 1
        self.score = 0
        self.lives = 6
        self.max_lives = 6
        self.hints_left = 3
        self.total_hints_used = 0
        self.multiplier = 15
        
        self.secret_word = ""
        self.category = ""
        self.guessed_word = []
        self.used_letters = set()
        self.used_words_pool = set()
        self.hint_used_this_round = False

        # Build Loading Screen Frame First
        self.create_loading_screen()

    def load_game_data(self):
        default_data = {"highscore": 0, "games_played": 0, "games_won": 0, "games_lost": 0}
        if not os.path.exists(DATA_FILE):
            return default_data
        try:
            with open(DATA_FILE, "r") as f:
                return json.load(f)
        except Exception:
            return default_data

    def save_game_data(self):
        try:
            with open(DATA_FILE, "w") as f:
                json.dump(self.system_profile_data, f, indent=4)
        except Exception as e:
            print(f"[Error saving data: {e}]")

    # ==============================================================================
    # ✨ ANIMATED LOADING SCREEN FRAME
    # ==============================================================================
    def create_loading_screen(self):
        self.loading_frame = tk.Frame(self.root, bg="#000000")
        self.loading_frame.pack(fill="both", expand=True)

        title_lbl = tk.Label(self.loading_frame, text="🎯 CONSOLE HANGMAN PRO 🎯", font=("Courier New", 24, "bold"), fg="#00E5FF", bg="#000000")
        title_lbl.pack(pady=(180, 20))

        self.msg_lbl = tk.Label(self.loading_frame, text="BOOT SEQUENCE INITIALIZED...", font=("Courier New", 12), fg="#FFFFFF", bg="#000000")
        self.msg_lbl.pack(pady=10)

        # High Contrast Saturated Cyan Progress Bar
        self.style.configure("Retro.Horizontal.TProgressbar", troughcolor="#111111", background="#00E5FF", thickness=15)
        self.progress = ttk.Progressbar(self.loading_frame, style="Retro.Horizontal.TProgressbar", orient="horizontal", length=500, mode="determinate")
        self.progress.pack(pady=20)

        self.root.after(100, self.animate_loading, 0)

    def animate_loading(self, value):
        if value <= 100:
            self.progress['value'] = value
            if value == 25:
                self.msg_lbl.config(text="LOADING HIGH-DPI CRT GRAPHICS MATRIX...")
            elif value == 55:
                self.msg_lbl.config(text="CONNECTING INTERNAL SYNTH AUDIO SOUNDS...")
            elif value == 80:
                self.msg_lbl.config(text="LOADING DICTIONARY KEYWORD ARRAYS...")
            self.root.after(12, self.animate_loading, value + 1)
        else:
            self.loading_frame.pack_forget()
            self.create_main_menu()

    # ==============================================================================
    # 📊 MAIN MENU SCREEN FRAME
    # ==============================================================================
    def create_main_menu(self):
        self.menu_frame = tk.Frame(self.root, bg="#000000")
        self.menu_frame.pack(fill="both", expand=True)

        # Title Layout
        banner = tk.Label(self.menu_frame, text="========================================\n  🎯   M A I N   M E N U   M O D U L E   🎯  \n========================================", font=("Courier New", 16, "bold"), fg="#00E5FF", bg="#000000", justify="center")
        banner.pack(pady=40)

        # Profile/Stats Card Display (White wireframe box on pitch black background)
        stats_box = tk.LabelFrame(self.menu_frame, text=" [ SYSTEM RECORD REGISTER ] ", font=("Courier New", 11, "bold"), fg="#FFFFFF", bg="#000000", padx=25, pady=20, bd=2, relief="solid")
        stats_box.pack(pady=10, fill="x", padx=120)

        tk.Label(stats_box, text=f"🏆 ALL-TIME HIGH SCORE : {self.system_profile_data['highscore']} PTS", font=("Courier New", 13, "bold"), fg="#FFC107", bg="#000000").pack(anchor="w", pady=4)
        tk.Label(stats_box, text=f"📊 TOTAL MATCHES RUN   : {self.system_profile_data['games_played']}", font=("Courier New", 11), fg="#FFFFFF", bg="#000000").pack(anchor="w", pady=4)
        tk.Label(stats_box, text=f"✅ SUCCESSFUL MISSIONS : {self.system_profile_data['games_won']}", font=("Courier New", 11), fg="#00E676", bg="#000000").pack(anchor="w", pady=4)
        tk.Label(stats_box, text=f"❌ SYSTEM DROPOUTS     : {self.system_profile_data['games_lost']}", font=("Courier New", 11), fg="#FF1744", bg="#000000").pack(anchor="w", pady=4)

        # Setup Selection Tier Action Interface
        btn_frame = tk.Frame(self.menu_frame, bg="#000000")
        btn_frame.pack(pady=40)

        play_btn = tk.Button(btn_frame, text="[ START SIMULATION ]", command=self.show_difficulty_menu, font=("Courier New", 13, "bold"), bg="#000000", fg="#00E676", width=26, activebackground="#00E676", activeforeground="#000000", bd=2, relief="solid", cursor="hand2", padx=10, pady=8)
        play_btn.pack(pady=12)

        exit_btn = tk.Button(btn_frame, text="[ TERMINATE PROGRAM ]", command=self.root.quit, font=("Courier New", 13, "bold"), bg="#000000", fg="#FF1744", width=26, activebackground="#FF1744", activeforeground="#000000", bd=2, relief="solid", cursor="hand2", padx=10, pady=8)
        exit_btn.pack(pady=5)

    # ==============================================================================
    # 🛠️ DIFFICULTY SELECTION SCREEN FRAME
    # ==============================================================================
    def show_difficulty_menu(self):
        self.menu_frame.pack_forget()
        self.diff_frame = tk.Frame(self.root, bg="#000000")
        self.diff_frame.pack(fill="both", expand=True)

        title = tk.Label(self.diff_frame, text="🛠️ SELECT DIFFICULTY CONFIGURATION", font=("Courier New", 16, "bold"), fg="#00E5FF", bg="#000000")
        title.pack(pady=60)

        for tier in ["easy", "normal", "hard"]:
            cfg = DIFFICULTY_CONFIG[tier]
            btn_text = f"[ {tier.upper()} // LIVES: {cfg['lives']} // SCORE MULTI: x{cfg['multiplier']} ]"
            btn_color = "#00E676" if tier == "easy" else "#FFC107" if tier == "normal" else "#FF1744"
            
            btn = tk.Button(self.diff_frame, text=btn_text, command=lambda t=tier: self.start_game_session(t), font=("Courier New", 12, "bold"), bg="#000000", fg=btn_color, width=48, activebackground=btn_color, activeforeground="#000000", bd=2, relief="solid", cursor="hand2", pady=12)
            btn.pack(pady=15)

    # ==============================================================================
    # 🎮 ACTIVE CORE MATCH GAMEPLAY FRAME
    # ==============================================================================
    def start_game_session(self, tier):
        self.diff_frame.pack_forget()
        
        # Configure Initial Match Variables
        self.selected_tier = tier
        self.max_lives = DIFFICULTY_CONFIG[tier]["lives"]
        self.multiplier = DIFFICULTY_CONFIG[tier]["multiplier"]
        
        self.current_round = 1
        self.score = 0
        self.hints_left = 3
        self.total_hints_used = 0
        self.used_words_pool.clear()

        # Build Interactive HUD Interface Layer
        self.create_gameplay_hud()
        self.setup_new_round()

    def create_gameplay_hud(self):
        self.game_frame = tk.Frame(self.root, bg="#000000")
        self.game_frame.pack(fill="both", expand=True)

        # Top Wireframe Grid HUD Info Bar
        self.hud_bar = tk.Frame(self.game_frame, bg="#000000", height=45, bd=2, relief="solid")
        self.hud_bar.pack(fill="x", side="top", padx=10, pady=10)

        self.lbl_round = tk.Label(self.hud_bar, text="", font=("Courier New", 12, "bold"), fg="#00E5FF", bg="#000000")
        self.lbl_round.pack(side="left", padx=25, pady=10)

        self.lbl_score = tk.Label(self.hud_bar, text="", font=("Courier New", 12, "bold"), fg="#FFC107", bg="#000000")
        self.lbl_score.pack(side="left", padx=45, pady=10)

        self.lbl_hints = tk.Label(self.hud_bar, text="", font=("Courier New", 12, "bold"), fg="#00E5FF", bg="#000000")
        self.lbl_hints.pack(side="right", padx=25, pady=10)

        # Main Performance Arena Base Layout Panels
        self.lbl_category = tk.Label(self.game_frame, text="", font=("Courier New", 14, "bold"), fg="#00E5FF", bg="#000000")
        self.lbl_category.pack(pady=(25, 5))

        self.lbl_hearts = tk.Label(self.game_frame, text="", font=("Courier New", 14), fg="#FF1744", bg="#000000")
        self.lbl_hearts.pack(pady=5)

        # Hidden Word Field Array View Layout (Ultra Bright Saturated Green)
        self.lbl_secret_word = tk.Label(self.game_frame, text="", font=("Courier New", 32, "bold"), fg="#00E676", bg="#000000")
        self.lbl_secret_word.pack(pady=35)

        # Dynamic Tracking Letters Output
        self.lbl_used_letters = tk.Label(self.game_frame, text="Guessed: None", font=("Courier New", 12), fg="#FFC107", bg="#000000")
        self.lbl_used_letters.pack(pady=5)

        # Interactive Input Mechanics Form Box Array
        input_container = tk.Frame(self.game_frame, bg="#000000")
        input_container.pack(pady=25)

        tk.Label(input_container, text="ENTER CHARACTER KEY: ", font=("Courier New", 11, "bold"), fg="#FFFFFF", bg="#000000").pack(side="left")
        
        self.entry_guess = tk.Entry(input_container, font=("Courier New", 14, "bold"), width=5, justify="center", bg="#000000", fg="#FFFFFF", insertbackground="white", bd=2, relief="solid")
        self.entry_guess.pack(side="left", padx=10)
        self.entry_guess.bind("<Return>", lambda event: self.process_guess_action())
        self.entry_guess.focus_set()

        self.btn_submit = tk.Button(input_container, text="[VERIFY]", command=self.process_guess_action, font=("Courier New", 11, "bold"), bg="#000000", fg="#00E5FF", activebackground="#00E5FF", activeforeground="#000000", bd=1, relief="solid", padx=15, pady=4, cursor="hand2")
        self.btn_submit.pack(side="left", padx=5)

        self.btn_hint = tk.Button(self.game_frame, text="[ DEPLOY HINT MODULE ]", command=self.trigger_hint_module, font=("Courier New", 11, "bold"), bg="#000000", fg="#00E5FF", activebackground="#00E5FF", activeforeground="#000000", bd=2, relief="solid", padx=18, pady=8, cursor="hand2")
        self.btn_hint.pack(pady=10)

    # ==============================================================================
    # ⚙️ LOGIC ENGINE OPERATIONS
    # ==============================================================================
    def setup_new_round(self):
        self.lives = self.max_lives
        self.hint_used_this_round = False
        self.used_letters.clear()

        available_pool = [
            (word, cat) for cat, words in WORDS_DICTIONARY.items()
            for word in words if word not in self.used_words_pool
        ]
        if not available_pool:
            available_pool = [(word, cat) for cat, words in WORDS_DICTIONARY.items() for word in words]

        self.secret_word, self.category = random.choice(available_pool)
        self.used_words_pool.add(self.secret_word)
        self.guessed_word = ["_"] * len(self.secret_word)

        self.refresh_hud_display()

    def refresh_hud_display(self):
        self.lbl_round.config(text=f"📋 ROUND: {self.current_round}/3")
        self.lbl_score.config(text=f"✨ SCORE: {self.score} PTS")
        self.lbl_hints.config(text=f"💡 HINTS: {self.hints_left}")
        
        self.lbl_category.config(text=f"📂 TARGET SECTOR: {self.category.upper()}")
        self.lbl_secret_word.config(text=" ".join(self.guessed_word))
        
        # Saturated vibrant native heart visuals
        hearts_str = "❤️ " * self.lives
        blacks_str = "🖤 " * (self.max_lives - self.lives)
        self.lbl_hearts.config(text=f"🩺 LIVES: {hearts_str}{blacks_str}({self.lives}/{self.max_lives})")

        used_str = ", ".join(sorted(list(self.used_letters))).upper() if self.used_letters else "NONE"
        self.lbl_used_letters.config(text=f"🛑 GUESSED LETTERS: [ {used_str} ]")

    def process_guess_action(self):
        user_input = self.entry_guess.get().strip().lower()
        self.entry_guess.delete(0, tk.END)

        if len(user_input) != 1 or not user_input.isalpha():
            AudioEngine.play_wrong()
            messagebox.showwarning("ERROR", "INVALID INPUT. PLEASE TYPE EXACTLY ONE LETTER VECTOR.")
            return

        if user_input in self.used_letters:
            AudioEngine.play_wrong()
            messagebox.showinfo("CACHE MATCH", f"THE LETTER '{user_input.upper()}' WAS ALREADY PROCESS CACHED.")
            return

        self.used_letters.add(user_input)

        if user_input in self.secret_word:
            AudioEngine.play_correct()
            for idx, char in enumerate(self.secret_word):
                if char == user_input:
                    self.guessed_word[idx] = user_input
        else:
            AudioEngine.play_wrong()
            self.lives -= 1

        self.refresh_hud_display()
        self.evaluate_match_state_nodes()

    def trigger_hint_module(self):
        if self.hint_used_this_round:
            messagebox.showwarning("ERROR", "A HINT LINK SECTOR HAS ALREADY BEEN GENERATED FOR THIS MATCH.")
            return
        if self.hints_left <= 0:
            messagebox.showwarning("ERROR", "NO REMAINING STORAGE BACKUP HINT MODULES FOUND.")
            return

        unrevealed_indices = [i for i, char in enumerate(self.guessed_word) if char == "_"]
        if unrevealed_indices:
            target_idx = random.choice(unrevealed_indices)
            chosen_letter = self.secret_word[target_idx]

            for i, char in enumerate(self.secret_word):
                if char == chosen_letter:
                    self.guessed_word[i] = chosen_letter

            self.used_letters.add(chosen_letter)
            self.hints_left -= 1
            self.total_hints_used += 1
            self.hint_used_this_round = True
            
            AudioEngine.play_hint()
            self.refresh_hud_display()
            self.evaluate_match_state_nodes()

    def evaluate_match_state_nodes(self):
        if "_" not in self.guessed_word:
            AudioEngine.play_win_round()
            self.score += self.multiplier
            messagebox.showinfo("SUCCESS", f"✨ TARGET WORD BYPASSED COMPLETELY:\n\n👉 [ {self.secret_word.upper()} ]")
            
            if self.current_round < 3:
                self.current_round += 1
                self.setup_new_round()
            else:
                self.finalize_game_match_metrics(won_all=True)
        
        elif self.lives <= 0:
            AudioEngine.play_game_over()
            messagebox.showerror("TERMINATED", f"☠️ CRITICAL MATRIX DECOUPLE!\n\nTHE WORD TARGET WAS: [ {self.secret_word.upper()} ]")
            self.finalize_game_match_metrics(won_all=False)

    # ==============================================================================
    # 🏆 END MATCH EVALUATION PERFORMANCE REPORTS
    # ==============================================================================
    def finalize_game_match_metrics(self, won_all):
        self.game_frame.pack_forget()

        # Save Metrics Data to System Engine Storage Profiles
        self.system_profile_data["games_played"] += 1
        if won_all and self.current_round == 3:
            self.system_profile_data["games_won"] += 1
        else:
            self.system_profile_data["games_lost"] += 1

        is_new_record = False
        if self.score > self.system_profile_data["highscore"]:
            self.system_profile_data["highscore"] = self.score
            is_new_record = True
        
        self.save_game_data()

        # Build Final Performance report text block exactly as requested [cite: 2]
        self.summary_frame = tk.Frame(self.root, bg="#000000")
        self.summary_frame.pack(fill="both", expand=True)

        print_banner = tk.Label(self.summary_frame, text="====================================\n\n        G A M E   S U M M A R Y\n\n====================================", font=("Courier New", 14, "bold"), fg="#00E5FF", bg="#000000")
        print_banner.pack(pady=25)

        # Clean Terminal Box Outline Panel Structure [cite: 2]
        card = tk.Frame(self.summary_frame, bg="#000000", padx=40, pady=25, bd=2, relief="solid")
        card.pack(pady=10)

        tk.Label(card, text=f"Score          : {self.score}", font=("Courier New", 13, "bold"), fg="#FFC107", bg="#000000").pack(anchor="w", pady=5)
        tk.Label(card, text=f"Rounds Won     : {self.current_round if won_all else self.current_round - 1}", font=("Courier New", 13, "bold"), fg="#FFFFFF", bg="#000000").pack(anchor="w", pady=5)
        tk.Label(card, text=f"Hints Used     : {self.total_hints_used}", font=("Courier New", 13, "bold"), fg="#FFFFFF", bg="#000000").pack(anchor="w", pady=5)
        tk.Label(card, text=f"High Score     : {self.system_profile_data['highscore']}", font=("Courier New", 13, "bold"), fg="#00E5FF", bg="#000000").pack(anchor="w", pady=5)

        evaluation_msg = "Excellent!" if won_all else "Mission Failed!"
        evaluation_color = "#00E676" if won_all else "#FF1744"
        
        eval_lbl = tk.Label(self.summary_frame, text=evaluation_msg, font=("Courier New", 16, "bold"), fg=evaluation_color, bg="#000000")
        eval_lbl.pack(pady=20)

        if is_new_record:
            rec_lbl = tk.Label(self.summary_frame, text="🏆 NEW HIGH SCORE OVERWRITTEN IN STORAGE! 🏆", font=("Courier New", 12, "bold"), fg="#00E676", bg="#000000")
            rec_lbl.pack(pady=5)
            AudioEngine.play_new_highscore()

        # Bottom Menu Control Return Navigation Interface Buttons
        menu_btn = tk.Button(self.summary_frame, text="[ RETURN TO MAIN SECTOR ]", command=self.exit_summary_to_menu, font=("Courier New", 12, "bold"), bg="#000000", fg="#00E5FF", activebackground="#00E5FF", activeforeground="#000000", bd=2, relief="solid", cursor="hand2", padx=20, pady=8)
        menu_btn.pack(pady=25)

    def exit_summary_to_menu(self):
        self.summary_frame.pack_forget()
        self.create_main_menu()

# ==============================================================================
# ENTRY POINT TRIGGER PIPELINE
# ==============================================================================
if __name__ == "__main__":
    window_root = tk.Tk()
    app = HangmanGUIApp(window_root)
    window_root.mainloop()
