import tkinter as tk
from tkinter import messagebox
from tkinter import scrolledtext
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from datetime import datetime

class SimpleMovieApp:
    def __init__(self, root):
        self.root = root
        self.root.title("My Movie Review App")
        self.root.geometry("1000x700")
        self.root.configure(bg='lightblue')
        
        self.reviews = []
        
        self.movies = [
            "Kantara", "KGF Chapter 2", "3 Idiots", "RRR", 
            "Avatar", "Avengers", "Spider-Man", "Bahubali",
            "Kirik Party", "Mungaru Male", "Lucia", "Ugramm",
            "The Dark Knight", "Inception", "Interstellar"
        ]
        
        self.quick_words = [
            "Awesome movie", "Great acting", "Nice story", "Good music",
            "Boring", "Slow story", "Bad acting", "Too long",
            "Funny", "Emotional", "Action packed", "Romantic",
            "Family movie", "Timepass", "Hit movie", "Flop movie"
        ]
        
        self.setup_gui()
    
    def setup_gui(self):
        title_label = tk.Label(
            self.root, 
            text="🎬 My Movie Review App 🍿",
            font=('Arial', 20, 'bold'),
            bg='lightblue',
            fg='darkblue'
        )
        title_label.pack(pady=10)
        
        notebook = tk.Frame(self.root, bg='lightblue')
        notebook.pack(fill='both', expand=True, padx=10, pady=10)
        
        self.create_input_section(notebook)
        self.create_analysis_section(notebook)
        self.create_history_section(notebook)
    
    def create_input_section(self, parent):
        input_frame = tk.LabelFrame(parent, text=" Write Your Review ", font=('Arial', 12, 'bold'), bg='lightblue')
        input_frame.pack(fill='x', padx=10, pady=5)
        
        movie_label = tk.Label(input_frame, text="Select Movie:", font=('Arial', 11), bg='lightblue')
        movie_label.grid(row=0, column=0, padx=10, pady=5, sticky='w')
        
        self.movie_var = tk.StringVar()
        self.movie_combo = tk.Entry(input_frame, textvariable=self.movie_var, width=30, font=('Arial', 11))
        self.movie_combo.grid(row=0, column=1, padx=10, pady=5)
        
        movie_suggest_label = tk.Label(input_frame, text="Or select from list:", font=('Arial', 10), bg='lightblue')
        movie_suggest_label.grid(row=1, column=0, padx=10, pady=2, sticky='w')
        
        movie_list_frame = tk.Frame(input_frame, bg='lightblue')
        movie_list_frame.grid(row=1, column=1, columnspan=2, padx=10, pady=2)
        
        for i, movie in enumerate(self.movies):
            btn = tk.Button(
                movie_list_frame,
                text=movie,
                command=lambda m=movie: self.select_movie(m),
                font=('Arial', 8),
                bg='lightyellow',
                width=12
            )
            btn.grid(row=i//6, column=i%6, padx=2, pady=2)
        
        rating_label = tk.Label(input_frame, text="Your Rating:", font=('Arial', 11), bg='lightblue')
        rating_label.grid(row=2, column=0, padx=10, pady=5, sticky='w')
        
        self.rating_var = tk.IntVar(value=5)
        rating_scale = tk.Scale(input_frame, from_=1, to=10, orient='horizontal', 
                               variable=self.rating_var, length=300, bg='lightblue')
        rating_scale.grid(row=2, column=1, padx=10, pady=5)
        
        quick_words_label = tk.Label(input_frame, text="Quick Feelings:", font=('Arial', 11), bg='lightblue')
        quick_words_label.grid(row=3, column=0, padx=10, pady=5, sticky='w')
        
        words_frame = tk.Frame(input_frame, bg='lightblue')
        words_frame.grid(row=3, column=1, columnspan=2, padx=10, pady=5)
        
        for i, word in enumerate(self.quick_words):
            btn = tk.Button(
                words_frame,
                text=word,
                command=lambda w=word: self.add_quick_word(w),
                font=('Arial', 8),
                bg='lightgreen',
                width=15
            )
            btn.grid(row=i//4, column=i%4, padx=2, pady=2)
        
        review_label = tk.Label(input_frame, text="Your Review:", font=('Arial', 11), bg='lightblue')
        review_label.grid(row=4, column=0, padx=10, pady=5, sticky='w')
        
        self.review_text = scrolledtext.ScrolledText(input_frame, width=60, height=8, font=('Arial', 10))
        self.review_text.grid(row=4, column=1, columnspan=2, padx=10, pady=5)
        
        button_frame = tk.Frame(input_frame, bg='lightblue')
        button_frame.grid(row=5, column=0, columnspan=3, pady=10)
        
        analyze_btn = tk.Button(
            button_frame,
            text="Analyze Review",
            command=self.analyze_review,
            font=('Arial', 12, 'bold'),
            bg='orange',
            fg='white',
            width=15
        )
        analyze_btn.pack(side='left', padx=10)
        
        clear_btn = tk.Button(
            button_frame,
            text="Clear",
            command=self.clear_all,
            font=('Arial', 12),
            bg='lightcoral',
            width=15
        )
        clear_btn.pack(side='left', padx=10)
    
    def create_analysis_section(self, parent):
        analysis_frame = tk.LabelFrame(parent, text=" Analysis Results ", font=('Arial', 12, 'bold'), bg='lightblue')
        analysis_frame.pack(fill='x', padx=10, pady=5)
        
        self.results_text = scrolledtext.ScrolledText(analysis_frame, width=80, height=12, font=('Arial', 10))
        self.results_text.pack(padx=10, pady=10)
        self.results_text.config(state='disabled')
    
    def create_history_section(self, parent):
        history_frame = tk.LabelFrame(parent, text=" Review History ", font=('Arial', 12, 'bold'), bg='lightblue')
        history_frame.pack(fill='both', expand=True, padx=10, pady=5)
        
        self.history_listbox = tk.Listbox(history_frame, font=('Arial', 10), height=8)
        self.history_listbox.pack(fill='both', expand=True, padx=10, pady=10)
        
        history_buttons = tk.Frame(history_frame, bg='lightblue')
        history_buttons.pack(pady=5)
        
        view_btn = tk.Button(
            history_buttons,
            text="View Selected",
            command=self.view_review,
            font=('Arial', 10),
            bg='lightgreen'
        )
        view_btn.pack(side='left', padx=5)
        
        delete_btn = tk.Button(
            history_buttons,
            text="Delete Selected",
            command=self.delete_review,
            font=('Arial', 10),
            bg='lightcoral'
        )
        delete_btn.pack(side='left', padx=5)
    
    def select_movie(self, movie_name):
        self.movie_var.set(movie_name)
    
    def add_quick_word(self, word):
        current_text = self.review_text.get("1.0", "end-1c")
        if current_text:
            new_text = current_text + ", " + word
        else:
            new_text = word
        self.review_text.delete("1.0", "end")
        self.review_text.insert("1.0", new_text)
    
    def analyze_review(self):
        movie_name = self.movie_var.get().strip()
        rating = self.rating_var.get()
        review_text = self.review_text.get("1.0", "end-1c").strip()
        
        if not movie_name:
            messagebox.showerror("Error", "Please enter a movie name!")
            return
        
        if not review_text:
            messagebox.showerror("Error", "Please write a review!")
            return
        
        sentiment = self.get_sentiment(review_text)
        
        review_data = {
            'movie': movie_name,
            'rating': rating,
            'review': review_text,
            'sentiment': sentiment,
            'time': datetime.now().strftime("%Y-%m-%d %H:%M")
        }
        
        self.reviews.append(review_data)
        self.show_results(review_data)
        self.update_history()
        
        messagebox.showinfo("Success", "Review analyzed successfully!")
    
    def get_sentiment(self, text):
        positive_words = ['good', 'great', 'awesome', 'amazing', 'excellent', 'best', 'love', 'nice', 'wonderful', 'fantastic']
        negative_words = ['bad', 'worst', 'terrible', 'awful', 'boring', 'hate', 'dislike', 'poor', 'waste']
        
        text_lower = text.lower()
        positive_count = sum(1 for word in positive_words if word in text_lower)
        negative_count = sum(1 for word in negative_words if word in text_lower)
        
        if positive_count > negative_count:
            return "Positive 😊"
        elif negative_count > positive_count:
            return "Negative 😞"
        else:
            return "Neutral 😐"
    
    def show_results(self, review):
        self.results_text.config(state='normal')
        self.results_text.delete("1.0", "end")
        
        result = f"""
MOVIE: {review['movie']}
RATING: {review['rating']}/10
SENTIMENT: {review['sentiment']}
TIME: {review['time']}

YOUR REVIEW:
{review['review']}

ANALYSIS:
"""
        if review['sentiment'] == "Positive 😊":
            result += "Your review sounds positive! People will like this movie."
        elif review['sentiment'] == "Negative 😞":
            result += "Your review sounds negative. You didn't like this movie much."
        else:
            result += "Your review sounds neutral. You have mixed feelings about this movie."
        
        self.results_text.insert("1.0", result)
        self.results_text.config(state='disabled')
    
    def update_history(self):
        self.history_listbox.delete(0, 'end')
        for i, review in enumerate(self.reviews):
            entry = f"{i+1}. {review['movie']} - {review['rating']}/10 - {review['sentiment']}"
            self.history_listbox.insert('end', entry)
    
    def view_review(self):
        selection = self.history_listbox.curselection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a review to view!")
            return
        
        index = selection[0]
        review = self.reviews[index]
        
        view_window = tk.Toplevel(self.root)
        view_window.title(f"Review: {review['movie']}")
        view_window.geometry("500x400")
        
        text = scrolledtext.ScrolledText(view_window, width=60, height=20, font=('Arial', 10))
        text.pack(padx=10, pady=10)
        
        content = f"""
Movie: {review['movie']}
Rating: {review['rating']}/10
Sentiment: {review['sentiment']}
Time: {review['time']}

Review:
{review['review']}
"""
        text.insert("1.0", content)
        text.config(state='disabled')
    
    def delete_review(self):
        selection = self.history_listbox.curselection()
        if not selection:
            messagebox.showwarning("Warning", "Please select a review to delete!")
            return
        
        index = selection[0]
        movie_name = self.reviews[index]['movie']
        
        if messagebox.askyesno("Confirm", f"Delete review for {movie_name}?"):
            del self.reviews[index]
            self.update_history()
            messagebox.showinfo("Success", "Review deleted!")
    
    def clear_all(self):
        self.movie_var.set("")
        self.rating_var.set(5)
        self.review_text.delete("1.0", "end")
        self.results_text.config(state='normal')
        self.results_text.delete("1.0", "end")
        self.results_text.config(state='disabled')

def main():
    root = tk.Tk()
    app = SimpleMovieApp(root)
    root.mainloop()

if __name__ == "__main__":
    main()
        
