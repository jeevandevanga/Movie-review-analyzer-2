# Mimport tkinter as tk
from tkinter import ttk, messagebox, scrolledtext
import pandas as pd
import numpy as np
from textblob import TextBlob
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from matplotlib.figure import Figure
import seaborn as sns
from collections import Counter
import re
from datetime import datetime
import json
import os

class MovieReviewAnalyzer:
    def __init__(self, root):
        self.root = root
        self.root.title("🎬 CineSense Pro - Advanced Movie Review Analyzer 🎭")
        self.root.geometry("1400x900")
        self.root.configure(bg='#1a1a2e')
        
        # Unique color scheme
        self.colors = {
            'primary': '#162447',
            'secondary': '#1f4068',
            'accent': '#e43f5a',
            'background': '#1a1a2e',
            'text': '#ffffff',
            'success': '#4CAF50',
            'warning': '#FF9800',
            'danger': '#f44336',
            'info': '#2196F3'
        }
        
        # Most used words in movie reviews with emojis
        self.quick_review_words = {
            'Positive': [
                "🎯 Amazing", "🌟 Brilliant", "👏 Masterpiece", "💫 Outstanding", 
                "😊 Enjoyable", "🤩 Spectacular", "💖 Heartwarming", "😂 Hilarious",
                "🎭 Powerful", "✨ Magical", "🏆 Award-worthy", "🚀 Thrilling",
                "🇮🇳 Desi Magic", "💃 Blockbuster", "🕺 Timepass", "🙏 Emotional"
            ],
            'Negative': [
                "😞 Disappointing", "😴 Boring", "👎 Poor", "💀 Terrible",
                "😤 Frustrating", "🤢 Awful", "💤 Sleep-inducing", "🚮 Wasteful",
                "😫 Painful", "🤦‍♂️ Cringey", "📉 Underwhelming", "💔 Heartbreaking",
                "💸 Time Waste", "📛 Flop Show", "🔇 Noise Pollution", "🤯 Confusing"
            ],
            'Descriptive': [
                "🎬 Visually stunning", "🎵 Great soundtrack", "🎭 Strong acting",
                "📖 Well-written", "🎪 Epic scale", "🔍 Thought-provoking",
                "💡 Original", "⚡ Fast-paced", "🌙 Atmospheric", "🎨 Artistic",
                "🔮 Mysterious", "❤️ Emotional", "🕌 Cultural", "🎪 Mass Entertainer",
                "💑 Romantic", "🔫 Action-packed", "😂 Comedy", "😭 Emotional Drama"
            ]
        }
        
        # Enhanced movie data with Indian and Kannada movies
        self.movies_data = {
            # Hollywood Classics
            'The Shawshank Redemption': {'genre': 'Drama', 'year': 1994, 'rating': 9.3, 'language': 'English'},
            'The Godfather': {'genre': 'Crime', 'year': 1972, 'rating': 9.2, 'language': 'English'},
            'The Dark Knight': {'genre': 'Action', 'year': 2008, 'rating': 9.0, 'language': 'English'},
            
            # Bollywood Movies
            '3 Idiots': {'genre': 'Comedy-Drama', 'year': 2009, 'rating': 8.4, 'language': 'Hindi'},
            'Dangal': {'genre': 'Sports Drama', 'year': 2016, 'rating': 8.3, 'language': 'Hindi'},
            'Lagaan': {'genre': 'Sports Drama', 'year': 2001, 'rating': 8.1, 'language': 'Hindi'},
            'Taare Zameen Par': {'genre': 'Drama', 'year': 2007, 'rating': 8.3, 'language': 'Hindi'},
            'Queen': {'genre': 'Comedy-Drama', 'year': 2013, 'rating': 8.2, 'language': 'Hindi'},
            'Gully Boy': {'genre': 'Musical Drama', 'year': 2019, 'rating': 8.0, 'language': 'Hindi'},
            'Bahubali: The Beginning': {'genre': 'Epic Action', 'year': 2015, 'rating': 8.0, 'language': 'Telugu'},
            'Bahubali: The Conclusion': {'genre': 'Epic Action', 'year': 2017, 'rating': 8.2, 'language': 'Telugu'},
            'RRR': {'genre': 'Action Drama', 'year': 2022, 'rating': 8.0, 'language': 'Telugu'},
            
            # Kannada Movies
            'Kantara': {'genre': 'Action Thriller', 'year': 2022, 'rating': 8.7, 'language': 'Kannada'},
            'KGF: Chapter 1': {'genre': 'Action', 'year': 2018, 'rating': 8.2, 'language': 'Kannada'},
            'KGF: Chapter 2': {'genre': 'Action', 'year': 2022, 'rating': 8.3, 'language': 'Kannada'},
            'Kirik Party': {'genre': 'Comedy Drama', 'year': 2016, 'rating': 8.4, 'language': 'Kannada'},
            'Mungaru Male': {'genre': 'Romance', 'year': 2006, 'rating': 8.2, 'language': 'Kannada'},
            'Ulidavaru Kandanthe': {'genre': 'Crime Thriller', 'year': 2014, 'rating': 8.3, 'language': 'Kannada'},
            'Lucia': {'genre': 'Psychological Thriller', 'year': 2013, 'rating': 8.5, 'language': 'Kannada'},
            'Googly': {'genre': 'Romance', 'year': 2013, 'rating': 8.1, 'language': 'Kannada'},
            'Ugramm': {'genre': 'Action Thriller', 'year': 2014, 'rating': 8.2, 'language': 'Kannada'},
            'RangiTaranga': {'genre': 'Mystery Thriller', 'year': 2015, 'rating': 8.6, 'language': 'Kannada'},
            
            # More Indian Movies
            'Vikram Vedha': {'genre': 'Action Thriller', 'year': 2017, 'rating': 8.4, 'language': 'Tamil'},
            'Super Deluxe': {'genre': 'Drama', 'year': 2019, 'rating': 8.3, 'language': 'Tamil'},
            'Drishyam': {'genre': 'Thriller', 'year': 2013, 'rating': 8.5, 'language': 'Malayalam'},
            'Premam': {'genre': 'Romance', 'year': 2015, 'rating': 8.3, 'language': 'Malayalam'},
            'Pelli Choopulu': {'genre': 'Romantic Comedy', 'year': 2016, 'rating': 8.3, 'language': 'Telugu'},
            'Jersey': {'genre': 'Sports Drama', 'year': 2019, 'rating': 8.5, 'language': 'Telugu'},
        }
        
        self.reviews = []
        self.selected_quick_words = []
        self.setup_ui()
        
    def setup_ui(self):
        # Create main frame
        main_frame = tk.Frame(self.root, bg=self.colors['background'])
        main_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)
        
        # Title with more emojis
        title_label = tk.Label(
            main_frame,
            text="🎬 CineSense Pro - Advanced Movie Review Analyzer 🎭",
            font=('Arial', 24, 'bold'),
            fg=self.colors['accent'],
            bg=self.colors['background']
        )
        title_label.pack(pady=(0, 20))
        
        # Subtitle with Indian cinema focus
        subtitle_label = tk.Label(
            main_frame,
            text="🌟 Featuring Hollywood, Bollywood & Sandalwood Movies 🌟",
            font=('Arial', 14, 'italic'),
            fg=self.colors['text'],
            bg=self.colors['background']
        )
        subtitle_label.pack(pady=(0, 20))
        
        # Create notebook for tabs
        self.notebook = ttk.Notebook(main_frame)
        self.notebook.pack(fill=tk.BOTH, expand=True)
        
        # Configure notebook style
        style = ttk.Style()
        style.configure('TNotebook', background=self.colors['primary'])
        style.configure('TNotebook.Tab', 
                       background=self.colors['secondary'],
                       foreground=self.colors['text'],
                       padding=[20, 10])
        
        # Create tabs
        self.create_input_tab()
        self.create_analysis_tab()
        self.create_visualization_tab()
        self.create_history_tab()
        self.create_movie_browser_tab()
        
    def create_movie_browser_tab(self):
        # Movie Browser Tab
        browser_frame = tk.Frame(self.notebook, bg=self.colors['primary'])
        self.notebook.add(browser_frame, text="🎞️ Movie Browser")
        
        # Filter frame
        filter_frame = tk.Frame(browser_frame, bg=self.colors['primary'])
        filter_frame.pack(fill=tk.X, padx=20, pady=10)
        
        # Language filter
        tk.Label(filter_frame, text="Filter by Language:", 
                font=('Arial', 12, 'bold'),
                fg=self.colors['text'],
                bg=self.colors['primary']).pack(side=tk.LEFT, padx=10)
        
        self.language_var = tk.StringVar(value="All")
        languages = ["All", "Kannada", "Hindi", "English", "Telugu", "Tamil", "Malayalam"]
        for lang in languages:
            rb = tk.Radiobutton(filter_frame, text=lang, variable=self.language_var, value=lang,
                               command=self.filter_movies,
                               bg=self.colors['primary'],
                               fg=self.colors['text'],
                               selectcolor=self.colors['secondary'],
                               font=('Arial', 10))
            rb.pack(side=tk.LEFT, padx=5)
        
        # Movie list frame
        list_frame = tk.Frame(browser_frame, bg=self.colors['primary'])
        list_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=10)
        
        # Treeview for movie list
        columns = ('Movie', 'Genre', 'Year', 'Rating', 'Language')
        self.movie_tree = ttk.Treeview(list_frame, columns=columns, show='headings', height=15)
        
        # Define headings
        for col in columns:
            self.movie_tree.heading(col, text=col)
            self.movie_tree.column(col, width=120)
        
        # Add scrollbar
        scrollbar = ttk.Scrollbar(list_frame, orient=tk.VERTICAL, command=self.movie_tree.yview)
        self.movie_tree.configure(yscrollcommand=scrollbar.set)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        self.movie_tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        
        # Populate movie list
        self.populate_movie_list()
        
    def populate_movie_list(self, language_filter="All"):
        # Clear existing items
        for item in self.movie_tree.get_children():
            self.movie_tree.delete(item)
            
        # Add movies based on filter
        for movie, details in self.movies_data.items():
            if language_filter == "All" or details['language'] == language_filter:
                self.movie_tree.insert('', tk.END, values=(
                    movie, 
                    details['genre'], 
                    details['year'],
                    f"⭐ {details['rating']}",
                    details['language']
                ))
    
    def filter_movies(self):
        self.populate_movie_list(self.language_var.get())
        
    def create_input_tab(self):
        # Input Tab
        input_frame = tk.Frame(self.notebook, bg=self.colors['primary'])
        self.notebook.add(input_frame, text="📝 Review Input")
        
        # Movie selection
        movie_frame = tk.Frame(input_frame, bg=self.colors['primary'])
        movie_frame.pack(fill=tk.X, padx=20, pady=10)
        
        tk.Label(movie_frame, text="🎭 Select Movie:", 
                font=('Arial', 12, 'bold'),
                fg=self.colors['text'],
                bg=self.colors['primary']).pack(side=tk.LEFT)
        
        self.movie_var = tk.StringVar()
        movie_combo = ttk.Combobox(movie_frame, 
                                  textvariable=self.movie_var,
                                  values=list(self.movies_data.keys()),
                                  width=40,
                                  font=('Arial', 11))
        movie_combo.pack(side=tk.LEFT, padx=10)
        movie_combo.set('Kantara')  # Default to a popular Kannada movie
        
        # Custom movie entry
        custom_frame = tk.Frame(input_frame, bg=self.colors['primary'])
        custom_frame.pack(fill=tk.X, padx=20, pady=10)
        
        tk.Label(custom_frame, text="🎬 Or Enter Custom Movie:", 
                font=('Arial', 12, 'bold'),
                fg=self.colors['text'],
                bg=self.colors['primary']).pack(side=tk.LEFT)
        
        self.custom_movie_var = tk.StringVar()
        custom_entry = tk.Entry(custom_frame, 
                               textvariable=self.custom_movie_var,
                               width=40,
                               font=('Arial', 11),
                               bg=self.colors['secondary'],
                               fg=self.colors['text'],
                               insertbackground=self.colors['text'])
        custom_entry.pack(side=tk.LEFT, padx=10)
        
        # Rating selection with emojis
        rating_frame = tk.Frame(input_frame, bg=self.colors['primary'])
        rating_frame.pack(fill=tk.X, padx=20, pady=10)
        
        tk.Label(rating_frame, text="⭐ Rating (1-10):", 
                font=('Arial', 12, 'bold'),
                fg=self.colors['text'],
                bg=self.colors['primary']).pack(side=tk.LEFT)
        
        self.rating_var = tk.IntVar(value=8)
        rating_scale = tk.Scale(rating_frame, 
                               from_=1, to=10,
                               orient=tk.HORIZONTAL,
                               variable=self.rating_var,
                               length=400,
                               bg=self.colors['primary'],
                               fg=self.colors['text'],
                               highlightbackground=self.colors['primary'],
                               troughcolor=self.colors['secondary'])
        rating_scale.pack(side=tk.LEFT, padx=10)
        
        # Quick review words section
        quick_words_frame = tk.Frame(input_frame, bg=self.colors['primary'])
        quick_words_frame.pack(fill=tk.X, padx=20, pady=10)
        
        tk.Label(quick_words_frame, text="🚀 Quick Review Words (Click to Add):", 
                font=('Arial', 12, 'bold'),
                fg=self.colors['text'],
                bg=self.colors['primary']).pack(anchor=tk.W)
        
        # Create quick word buttons
        self.create_quick_word_buttons(quick_words_frame)
        
        # Selected words display
        selected_frame = tk.Frame(input_frame, bg=self.colors['primary'])
        selected_frame.pack(fill=tk.X, padx=20, pady=5)
        
        tk.Label(selected_frame, text="📋 Selected Words:", 
                font=('Arial', 11, 'bold'),
                fg=self.colors['text'],
                bg=self.colors['primary']).pack(anchor=tk.W)
        
        self.selected_words_label = tk.Label(selected_frame, 
                                           text="No words selected",
                                           font=('Arial', 10),
                                           fg=self.colors['accent'],
                                           bg=self.colors['primary'],
                                           wraplength=800)
        self.selected_words_label.pack(anchor=tk.W)
        
        # Review text area
        review_frame = tk.Frame(input_frame, bg=self.colors['primary'])
        review_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=10)
        
        tk.Label(review_frame, text="📝 Your Review:", 
                font=('Arial', 12, 'bold'),
                fg=self.colors['text'],
                bg=self.colors['primary']).pack(anchor=tk.W)
        
        self.review_text = scrolledtext.ScrolledText(
            review_frame,
            wrap=tk.WORD,
            width=80,
            height=12,
            font=('Arial', 11),
            bg=self.colors['secondary'],
            fg=self.colors['text'],
            insertbackground=self.colors['text']
        )
        self.review_text.pack(fill=tk.BOTH, expand=True, pady=5)
        
        # Buttons frame
        button_frame = tk.Frame(input_frame, bg=self.colors['primary'])
        button_frame.pack(fill=tk.X, padx=20, pady=20)
        
        analyze_btn = tk.Button(
            button_frame,
            text="🔍 Analyze Review",
            command=self.analyze_review,
            font=('Arial', 12, 'bold'),
            bg=self.colors['accent'],
            fg='white',
            padx=20,
            pady=10,
            cursor='hand2'
        )
        analyze_btn.pack(side=tk.LEFT, padx=10)
        
        clear_btn = tk.Button(
            button_frame,
            text="🗑️ Clear All",
            command=self.clear_input,
            font=('Arial', 12),
            bg=self.colors['secondary'],
            fg=self.colors['text'],
            padx=20,
            pady=10
        )
        clear_btn.pack(side=tk.LEFT, padx=10)
        
        # Sample reviews button with Indian focus
        sample_btn = tk.Button(
            button_frame,
            text="📋 Load Sample Reviews",
            command=self.load_sample_reviews,
            font=('Arial', 12),
            bg=self.colors['success'],
            fg='white',
            padx=20,
            pady=10
        )
        sample_btn.pack(side=tk.RIGHT, padx=10)
        
    def create_quick_word_buttons(self, parent):
        # Create frames for each category
        for category, words in self.quick_review_words.items():
            category_frame = tk.Frame(parent, bg=self.colors['primary'])
            category_frame.pack(fill=tk.X, pady=5)
            
            # Category label
            emoji = "👍" if category == "Positive" else "👎" if category == "Negative" else "📝"
            tk.Label(category_frame, text=f"{emoji} {category}:", 
                    font=('Arial', 10, 'bold'),
                    fg=self.colors['text'],
                    bg=self.colors['primary']).pack(anchor=tk.W)
            
            # Words frame
            words_frame = tk.Frame(category_frame, bg=self.colors['primary'])
            words_frame.pack(fill=tk.X, padx=20, pady=5)
            
            # Create buttons for each word
            for i, word in enumerate(words):
                btn = tk.Button(
                    words_frame,
                    text=word,
                    command=lambda w=word: self.add_quick_word(w),
                    font=('Arial', 9),
                    bg=self.colors['secondary'],
                    fg=self.colors['text'],
                    relief=tk.RAISED,
                    bd=2,
                    padx=8,
                    pady=4,
                    cursor='hand2'
                )
                btn.pack(side=tk.LEFT, padx=2, pady=2)
    
    def add_quick_word(self, word):
        # Remove emoji for processing, keep for display
        clean_word = re.sub(r'[^\w\s]', '', word).strip()
        if clean_word not in self.selected_quick_words:
            self.selected_quick_words.append(clean_word)
            
        # Update display
        display_text = " | ".join([w for w in self.quick_review_words['Positive'] + 
                                 self.quick_review_words['Negative'] + 
                                 self.quick_review_words['Descriptive'] 
                                 if re.sub(r'[^\w\s]', '', w).strip() in self.selected_quick_words])
        
        if not display_text:
            display_text = "No words selected"
            
        self.selected_words_label.config(text=display_text)
        
        # Add to review text
        current_text = self.review_text.get("1.0", tk.END).strip()
        if current_text:
            new_text = current_text + ", " + clean_word
        else:
            new_text = clean_word
            
        self.review_text.delete("1.0", tk.END)
        self.review_text.insert("1.0", new_text)
    
    def create_analysis_tab(self):
        # Analysis Tab
        analysis_frame = tk.Frame(self.notebook, bg=self.colors['primary'])
        self.notebook.add(analysis_frame, text="📊 Analysis Results")
        
        # Results display area
        self.results_text = scrolledtext.ScrolledText(
            analysis_frame,
            wrap=tk.WORD,
            width=80,
            height=25,
            font=('Arial', 11),
            bg=self.colors['secondary'],
            fg=self.colors['text'],
            state=tk.DISABLED
        )
        self.results_text.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)
        
    def create_visualization_tab(self):
        # Visualization Tab
        viz_frame = tk.Frame(self.notebook, bg=self.colors['primary'])
        self.notebook.add(viz_frame, text="📈 Visualizations")
        
        # Canvas for matplotlib figures
        self.viz_canvas_frame = tk.Frame(viz_frame, bg=self.colors['primary'])
        self.viz_canvas_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)
        
    def create_history_tab(self):
        # History Tab
        history_frame = tk.Frame(self.notebook, bg=self.colors['primary'])
        self.notebook.add(history_frame, text="📚 Review History")
        
        # History listbox
        history_list_frame = tk.Frame(history_frame, bg=self.colors['primary'])
        history_list_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=10)
        
        self.history_listbox = tk.Listbox(
            history_list_frame,
            font=('Arial', 11),
            bg=self.colors['secondary'],
            fg=self.colors['text'],
            selectbackground=self.colors['accent']
        )
        self.history_listbox.pack(fill=tk.BOTH, expand=True, side=tk.LEFT)
        
        # Scrollbar for listbox
        scrollbar = tk.Scrollbar(history_list_frame, orient=tk.VERTICAL)
        scrollbar.config(command=self.history_listbox.yview)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        self.history_listbox.config(yscrollcommand=scrollbar.set)
        
        # History buttons
        history_btn_frame = tk.Frame(history_frame, bg=self.colors['primary'])
        history_btn_frame.pack(fill=tk.X, padx=20, pady=10)
        
        view_btn = tk.Button(
            history_btn_frame,
            text="👁️ View Selected",
            command=self.view_selected_review,
            font=('Arial', 11),
            bg=self.colors['accent'],
            fg='white',
            padx=15,
            pady=5
        )
        view_btn.pack(side=tk.LEFT, padx=5)
        
        delete_btn = tk.Button(
            history_btn_frame,
            text="🗑️ Delete Selected",
            command=self.delete_selected_review,
            font=('Arial', 11),
            bg=self.colors['danger'],
            fg='white',
            padx=15,
            pady=5
        )
        delete_btn.pack(side=tk.LEFT, padx=5)
        
        export_btn = tk.Button(
            history_btn_frame,
            text="💾 Export History",
            command=self.export_history,
            font=('Arial', 11),
            bg=self.colors['success'],
            fg='white',
            padx=15,
            pady=5
        )
        export_btn.pack(side=tk.RIGHT, padx=5)
        
    def analyze_review(self):
        movie_name = self.movie_var.get()
        custom_movie = self.custom_movie_var.get().strip()
        rating = self.rating_var.get()
        review_text = self.review_text.get("1.0", tk.END).strip()
        
        # Use custom movie name if provided
        if custom_movie:
            movie_name = custom_movie
        
        if not movie_name:
            messagebox.showerror("Error", "🎬 Please select or enter a movie name!")
            return
            
        if not review_text:
            messagebox.showerror("Error", "📝 Please enter a review!")
            return
        
        # Perform sentiment analysis
        analysis = self.perform_sentiment_analysis(review_text)
        
        # Create review object
        review = {
            'movie': movie_name,
            'rating': rating,
            'review': review_text,
            'sentiment': analysis['sentiment'],
            'polarity': analysis['polarity'],
            'subjectivity': analysis['subjectivity'],
            'timestamp': datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            'keywords': analysis['keywords'],
            'language': self.movies_data.get(movie_name, {}).get('language', 'Unknown'),
            'quick_words': self.selected_quick_words.copy()
        }
        
        self.reviews.append(review)
        self.display_analysis_results(review)
        self.update_history_list()
        self.update_visualizations()
        
        messagebox.showinfo("Success", "✅ Review analyzed successfully!")
        
    def perform_sentiment_analysis(self, text):
        blob = TextBlob(text)
        polarity = blob.sentiment.polarity
        subjectivity = blob.sentiment.subjectivity
        
        # Determine sentiment
        if polarity > 0.1:
            sentiment = "Positive 👍"
            sentiment_emoji = "😊"
        elif polarity < -0.1:
            sentiment = "Negative 👎"
            sentiment_emoji = "😞"
        else:
            sentiment = "Neutral 😐"
            sentiment_emoji = "😐"
            
        # Extract keywords (simple approach)
        words = re.findall(r'\b[a-zA-Z]{4,}\b', text.lower())
        common_words = ['this', 'that', 'with', 'from', 'they', 'what', 'were', 'when', 'have', 'like']
        keywords = [word for word in words if word not in common_words][:10]
        
        return {
            'sentiment': f"{sentiment} {sentiment_emoji}",
            'polarity': polarity,
            'subjectivity': subjectivity,
            'keywords': keywords
        }
        
    def display_analysis_results(self, review):
        self.results_text.config(state=tk.NORMAL)
        self.results_text.delete(1.0, tk.END)
        
        # Get movie language
        movie_language = self.movies_data.get(review['movie'], {}).get('language', 'Unknown')
        language_emoji = "🇮🇳" if movie_language in ['Hindi', 'Kannada', 'Tamil', 'Telugu', 'Malayalam'] else "🌍"
        
        # Format results with enhanced emojis
        results = f"""
╔{'═'*70}╗
║{'🎬 CineSense Pro - Analysis Results 🎭':^70}║
╠{'═'*70}╣
║ {'🎭 Movie:':<15} {review['movie']:<53} ║
║ {'🏷️ Language:':<15} {movie_language} {language_emoji:<49} ║
║ {'⭐ Rating:':<15} {'★' * review['rating'] + '☆' * (10-review['rating']):<53} ║
║ {'📊 Sentiment:':<15} {review['sentiment']:<53} ║
╠{'═'*70}╣
║ {'📈 Detailed Analysis':^70} ║
╠{'═'*70}╣
║ {'🎯 Polarity Score:':<20} {review['polarity']:>8.3f} {' ':38}║
║ {'🔍 Subjectivity:':<20} {review['subjectivity']:>8.3f} {' ':38}║
╠{'═'*70}╣
║ {'💡 Key Insights:':<70} ║
╠{'═'*70}╣
"""
        
        # Add polarity interpretation
        polarity = review['polarity']
        if polarity > 0.7:
            polarity_text = "🎉 Extremely Positive - Highly enthusiastic review!"
        elif polarity > 0.3:
            polarity_text = "😊 Positive - Generally favorable opinion"
        elif polarity > 0.1:
            polarity_text = "🙂 Slightly Positive - Mildly positive with some reservations"
        elif polarity > -0.1:
            polarity_text = "😐 Neutral - Balanced or factual review"
        elif polarity > -0.3:
            polarity_text = "😟 Slightly Negative - Mild criticism"
        elif polarity > -0.7:
            polarity_text = "😞 Negative - Generally unfavorable"
        else:
            polarity_text = "😠 Extremely Negative - Strong criticism"
            
        results += f"║ {'• Polarity:':<15} {polarity_text:<53} ║\n"
        
        # Add subjectivity interpretation
        subjectivity = review['subjectivity']
        if subjectivity > 0.7:
            subj_text = "💭 Highly Subjective - Strong personal opinions"
        elif subjectivity > 0.4:
            subj_text = "📝 Moderately Subjective - Mix of facts and opinions"
        else:
            subj_text = "📊 Objective - Fact-based and analytical"
            
        results += f"║ {'• Subjectivity:':<15} {subj_text:<53} ║\n"
        
        # Add quick words if any
        if review['quick_words']:
            results += f"╠{'═'*70}╣\n"
            results += f"║ {'🚀 Quick Words Used:':<70} ║\n"
            quick_words_text = ' | '.join(review['quick_words'])
            # Split long text into multiple lines
            if len(quick_words_text) > 68:
                words_lines = [quick_words_text[i:i+68] for i in range(0, len(quick_words_text), 68)]
                for line in words_lines:
                    results += f"║ {line:<70} ║\n"
            else:
                results += f"║ {quick_words_text:<70} ║\n"
        
        # Add keywords
        results += f"╠{'═'*70}╣\n"
        results += f"║ {'🔑 Keywords Found:':<70} ║\n"
        keywords_text = ', '.join(review['keywords'][:8])
        results += f"║ {keywords_text:<70} ║\n"
        
        # Add review summary
        results += f"╠{'═'*70}╣\n"
        results += f"║ {'📝 Review Summary:':<70} ║\n"
        results += f"╠{'═'*70}╣\n"
        
        # Truncate review for display
        short_review = review['review'][:400] + "..." if len(review['review']) > 400 else review['review']
        # Split long review into multiple lines
        review_lines = [short_review[i:i+68] for i in range(0, len(short_review), 68)]
        for line in review_lines:
            results += f"║ {line:<70} ║\n"
        
        results += f"╚{'═'*70}╝\n"
        
        self.results_text.insert(tk.END, results)
        self.results_text.config(state=tk.DISABLED)
        
    def update_visualizations(self):
        # Clear previous visualizations
        for widget in self.viz_canvas_frame.winfo_children():
            widget.destroy()
            
        if not self.reviews:
            return
            
        # Create matplotlib figures
        fig = Figure(figsize=(14, 10), facecolor=self.colors['primary'])
        fig.suptitle('🎬 Review Analysis Dashboard', color=self.colors['text'], fontsize=16, fontweight='bold')
        
        # Create subplots
        gs = fig.add_gridspec(2, 3)
        ax1 = fig.add_subplot(gs[0, 0])  # Sentiment pie chart
        ax2 = fig.add_subplot(gs[0, 1])  # Rating distribution
        ax3 = fig.add_subplot(gs[0, 2])  # Language distribution
        ax4 = fig.add_subplot(gs[1, :])  # Polarity vs Subjectivity
        
        # Set background colors
        for ax in [ax1, ax2, ax3, ax4]:
            ax.set_facecolor(self.colors['secondary'])
            ax.tick_params(colors=self.colors['text'])
            for spine in ax.spines.values():
                spine.set_color(self.colors['text'])
        
        # Plot 1: Sentiment distribution
        sentiments = [review['sentiment'].split()[0] for review in self.reviews]
        sentiment_counts = Counter(sentiments)
        
        colors = [self.colors['success'], self.colors['warning'], self.colors['danger']]
        ax1.pie(sentiment_counts.values(), labels=sentiment_counts.keys(), autopct='%1.1f%%',
                colors=colors, textprops={'color': self.colors['text']})
        ax1.set_title('📊 Sentiment Distribution', color=self.colors['text'], fontweight='bold')
        
        # Plot 2: Rating distribution
        ratings = [review['rating'] for review in self.reviews]
        ax2.hist(ratings, bins=10, range=(1, 11), color=self.colors['accent'], alpha=0.7, edgecolor='white')
        ax2.set_title('⭐ Rating Distribution', color=self.colors['text'], fontweight='bold')
        ax2.set_xlabel('Rating', color=self.colors['text'])
        ax2.set_ylabel('Frequency', color=self.colors['text'])
        ax2.set_xticks(range(1, 11))
        
        # Plot 3: Language distribution
        languages = [review.get('language', 'Unknown') for review in self.reviews]
        language_counts = Counter(languages)
        ax3.bar(language_counts.keys(), language_counts.values(), color=self.colors['info'], alpha=0.7)
        ax3.set_title('🌍 Language Distribution', color=self.colors['text'], fontweight='bold')
        ax3.set_xlabel('Language', color=self.colors['text'])
        ax3.set_ylabel('Count', color=self.colors['text'])
        ax3.tick_params(axis='x', rotation=45)
        
        # Plot 4: Polarity vs Subjectivity scatter plot
        polarities = [review['polarity'] for review in self.reviews]
        subjectivities = [review['subjectivity'] for review in self.reviews]
        movie_names = [review['movie'] for review in self.reviews]
        ratings = [review['rating'] for review in self.reviews]
        
        scatter = ax4.scatter(polarities, subjectivities, c=ratings, cmap='viridis', s=100, alpha=0.7)
        ax4.set_title('📈 Polarity vs Subjectivity', color=self.colors['text'], fontweight='bold')
        ax4.set_xlabel('Polarity (Negative ← → Positive)', color=self.colors['text'])
        ax4.set_ylabel('Subjectivity (Objective ← → Subjective)', color=self.colors['text'])
        
        # Add colorbar
        cbar = fig.colorbar(scatter, ax=ax4)
        cbar.set_label('Rating', color=self.colors['text'])
        cbar.ax.yaxis.set_tick_params(color=self.colors['text'])
        cbar.outline.set_edgecolor(self.colors['text'])
        plt.setp(plt.getp(cbar.ax.axes, 'yticklabels'), color=self.colors['text'])
        
        # Add some annotations
        ax4.axvline(x=0, color='white', linestyle='--', alpha=0.5)
        ax4.axhline(y=0.5, color='white', linestyle='--', alpha=0.5)
        
        # Adjust layout
        fig.tight_layout(rect=[0, 0, 1, 0.95])
        
        # Embed in tkinter
        canvas = FigureCanvasTkAgg(fig, self.viz_canvas_frame)
        canvas.draw()
        canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True)
        
    def update_history_list(self):
        self.history_listbox.delete(0, tk.END)
        for i, review in enumerate(self.reviews):
            lang_emoji = "🇮🇳" if review.get('language') in ['Hindi', 'Kannada', 'Tamil', 'Telugu', 'Malayalam'] else "🌍"
            display_text = f"{i+1}. {review['movie']} {lang_emoji} - Rating: {review['rating']}/10 - {review['sentiment']} - {review['timestamp']}"
            self.history_listbox.insert(tk.END, display_text)
            
    def view_selected_review(self):
        selection = self.history_listbox.curselection()
        if not selection:
            messagebox.showwarning("Warning", "⚠️ Please select a review from the history!")
            return
            
        index = selection[0]
        review = self.reviews[index]
        
        # Create detailed view window
        detail_window = tk.Toplevel(self.root)
        detail_window.title(f"🎬 Review Details - {review['movie']}")
        detail_window.geometry("700x600")
        detail_window.configure(bg=self.colors['background'])
        
        # Display review details
        details_text = scrolledtext.ScrolledText(
            detail_window,
            wrap=tk.WORD,
            width=80,
            height=35,
            font=('Arial', 11),
            bg=self.colors['secondary'],
            fg=self.colors['text']
        )
        details_text.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)
        
        lang_emoji = "🇮🇳" if review.get('language') in ['Hindi', 'Kannada', 'Tamil', 'Telugu', 'Malayalam'] else "🌍"
        
        details = f"""
🎬 MOVIE: {review['movie']}
🏷️ LANGUAGE: {review.get('language', 'Unknown')} {lang_emoji}
⭐ RATING: {review['rating']}/10
📊 SENTIMENT: {review['sentiment']}
🎯 POLARITY: {review['polarity']:.3f}
🔍 SUBJECTIVITY: {review['subjectivity']:.3f}
⏰ TIMESTAMP: {review['timestamp']}

{'='*60}

📝 FULL REVIEW:
{'-'*50}
{review['review']}

{'='*60}

🔑 KEYWORDS: {', '.join(review['keywords'])}

"""
        
        if review['quick_words']:
            details += f"🚀 QUICK WORDS: {' | '.join(review['quick_words'])}\n"
            
        details_text.insert(tk.END, details)
        details_text.config(state=tk.DISABLED)
        
    def delete_selected_review(self):
        selection = self.history_listbox.curselection()
        if not selection:
            messagebox.showwarning("Warning", "⚠️ Please select a review to delete!")
            return
            
        index = selection[0]
        movie_name = self.reviews[index]['movie']
        
        if messagebox.askyesno("Confirm Delete", f"🗑️ Are you sure you want to delete the review for '{movie_name}'?"):
            del self.reviews[index]
            self.update_history_list()
            self.update_visualizations()
            messagebox.showinfo("Success", "✅ Review deleted successfully!")
            
    def export_history(self):
        if not self.reviews:
            messagebox.showwarning("Warning", "⚠️ No reviews to export!")
            return
            
        # Create export data
        export_data = {
            'export_date': datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            'total_reviews': len(self.reviews),
            'reviews': self.reviews
        }
        
        # Save to JSON file
        filename = f"movie_reviews_export_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        try:
            with open(filename, 'w', encoding='utf-8') as f:
                json.dump(export_data, f, indent=2, ensure_ascii=False)
            messagebox.showinfo("Success", f"✅ Reviews exported successfully to {filename}!")
        except Exception as e:
            messagebox.showerror("Error", f"❌ Failed to export reviews: {str(e)}")
            
    def clear_input(self):
        self.review_text.delete("1.0", tk.END)
        self.custom_movie_var.set("")
        self.rating_var.set(8)
        self.selected_quick_words.clear()
        self.selected_words_label.config(text="No words selected")
        
    def load_sample_reviews(self):
        sample_reviews = [
            {
                'movie': 'Kantara',
                'rating': 9,
                'review': 'An absolute masterpiece of Indian cinema! The storytelling is impeccable, the visuals are stunning, and the cultural elements are beautifully portrayed. Rishab Shetty delivers a career-defining performance.',
                'sentiment': 'Positive 👍 😊',
                'polarity': 0.8,
                'subjectivity': 0.7,
                'timestamp': datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                'keywords': ['masterpiece', 'storytelling', 'visuals', 'cultural', 'performance'],
                'language': 'Kannada',
                'quick_words': ['Amazing', 'Masterpiece', 'Visually stunning', 'Cultural']
            },
            {
                'movie': 'KGF: Chapter 2',
                'rating': 8,
                'review': 'Yash rocks as Rocky Bhai! The action sequences are spectacular and the scale is epic. However, the storyline feels slightly repetitive from the first part. Still a mass entertainer!',
                'sentiment': 'Positive 👍 😊',
                'polarity': 0.6,
                'subjectivity': 0.6,
                'timestamp': datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                'keywords': ['action', 'sequences', 'spectacular', 'epic', 'storyline'],
                'language': 'Kannada',
                'quick_words': ['Spectacular', 'Action-packed', 'Mass Entertainer']
            },
            {
                'movie': '3 Idiots',
                'rating': 9,
                'review': 'A perfect blend of entertainment and social message. Aamir Khan is brilliant and the film makes you laugh while making important points about education system.',
                'sentiment': 'Positive 👍 😊',
                'polarity': 0.7,
                'subjectivity': 0.65,
                'timestamp': datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                'keywords': ['entertainment', 'social', 'message', 'education', 'system'],
                'language': 'Hindi',
                'quick_words': ['Brilliant', 'Hilarious', 'Thought-provoking']
            }
        ]
        
        self.reviews.extend(sample_reviews)
        
