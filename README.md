[tictactoe.py](https://github.com/user-attachments/files/32658734/tictactoe.py)
# krestiki-noliki
крестики нолики на питоне с непобедимым ии
# -*- coding: utf-8 -*-
"""
Крестики-нолики с ИИ (минимакс) на tkinter.
Режимы: человек vs ИИ (3 уровня), человек vs человек.
Ты играешь крестиками (X) и ходишь первым.
"""

import random
import tkinter as tk
from tkinter import messagebox

EMPTY, X, O = "", "X", "O"

LEVELS = {
    "Лёгкий": 0.3,   # вероятность случайного хода
    "Средний": 0.7,
    "Непобедимый": 1.0,  # всегда лучший ход
}

WIN_LINES = [
    (0, 1, 2), (3, 4, 5), (6, 7, 8),  # строки
    (0, 3, 6), (1, 4, 7), (2, 5, 8),  # столбцы
    (0, 4, 8), (2, 4, 6),             # диагонали
]


def winner(board):
    """Возвращает (X/O, линия) при победе, ('draw', None) при ничьей, иначе (None, None)."""
    for a, b, c in WIN_LINES:
        if board[a] != EMPTY and board[a] == board[b] == board[c]:
            return board[a], (a, b, c)
    if EMPTY not in board:
        return "draw", None
    return None, None


def minimax(board, is_maximizing, depth=0):
    """Минимакс: ИИ играет за O. Возвращает оценку позиции."""
    w, _ = winner(board)
    if w == O:
        return 10 - depth
    if w == X:
        return depth - 10
    if w == "draw":
        return 0

    if is_maximizing:
        best = -float("inf")
        for i in range(9):
            if board[i] == EMPTY:
                board[i] = O
                best = max(best, minimax(board, False, depth + 1))
                board[i] = EMPTY
        return best
    else:
        best = float("inf")
        for i in range(9):
            if board[i] == EMPTY:
                board[i] = X
                best = min(best, minimax(board, True, depth + 1))
                board[i] = EMPTY
        return best


def best_move(board):
    """Лучший ход для ИИ (O)."""
    best_score, moves = -float("inf"), []
    for i in range(9):
        if board[i] == EMPTY:
            board[i] = O
            score = minimax(board, False)
            board[i] = EMPTY
            if score > best_score:
                best_score, moves = score, [i]
            elif score == best_score:
                moves.append(i)
    return random.choice(moves)


class TicTacToe:
    def __init__(self, root: tk.Tk):
        self.root = root
        root.title("Крестики-нолики")
        root.resizable(False, False)

        # Меню
        menubar = tk.Menu(root)
        mode_menu = tk.Menu(menubar, tearoff=0)
        for name in LEVELS:
            mode_menu.add_command(label=f"Против ИИ: {name}",
                                  command=lambda n=name: self.set_mode(n))
        mode_menu.add_command(label="Два игрока",
                              command=lambda: self.set_mode(None))
        mode_menu.add_separator()
        mode_menu.add_command(label="Выход", command=root.destroy)
        menubar.add_cascade(label="Режим", menu=mode_menu)
        root.config(menu=menubar)

        # Верхняя панель: статус и счёт
        top = tk.Frame(root, bg="#C0C0C0")
        top.pack(fill=tk.X, padx=6, pady=6)

        self.status_var = tk.StringVar()
        tk.Label(top, textvariable=self.status_var, font=("Segoe UI", 12, "bold"),
                 bg="#C0C0C0").pack(side=tk.LEFT)

        self.score_var = tk.StringVar()
        tk.Label(top, textvariable=self.score_var, font=("Segoe UI", 12),
                 bg="#C0C0C0").pack(side=tk.RIGHT)

        # Поле
        self.board_frame = tk.Frame(root, bg="#C0C0C0")
        self.board_frame.pack(padx=6, pady=6)

        self.buttons = []
        for i in range(9):
            b = tk.Button(self.board_frame, text="", width=4, height=2,
                          font=("Segoe UI", 24, "bold"), bg="#F0F0F0",
                          command=lambda i=i: self.on_click(i))
            b.grid(row=i // 3, column=i % 3, padx=2, pady=2)
            self.buttons.append(b)

        tk.Button(root, text="Новая игра", font=("Segoe UI", 11),
                  command=self.new_game).pack(pady=(0, 8))

        self.score = {"X": 0, "O": 0, "draw": 0}
        self.ai_level = "Непобедимый"
        self.vs_ai = True
        self.new_game()

    # ---------- Игра ----------

    def set_mode(self, level):
        if level is None:
            self.vs_ai = False
        else:
            self.vs_ai = True
            self.ai_level = level
        self.score = {"X": 0, "O": 0, "draw": 0}
        self.new_game()

    def new_game(self):
        self.board = [EMPTY] * 9
        self.current = X
        self.game_over = False
        for b in self.buttons:
            b.config(text="", bg="#F0F0F0", state=tk.NORMAL)
        self.update_labels()

    def update_labels(self):
        if self.game_over:
            return
        if self.vs_ai:
            who = "Твой ход (X)" if self.current == X else "Ходит ИИ…"
            mode = f"ИИ: {self.ai_level}"
        else:
            who = f"Ходит {self.current}"
            mode = "Два игрока"
        self.status_var.set(f"{mode}  |  {who}")
        self.score_var.set(
            f"X: {self.score['X']}   O: {self.score['O']}   Ничьи: {self.score['draw']}"
        )

    def on_click(self, i):
        if self.game_over or self.board[i] != EMPTY:
            return
        if self.vs_ai and self.current == O:
            return  # не даём кликать во время хода ИИ

        self.make_move(i)
        if not self.game_over and self.vs_ai and self.current == O:
            self.update_labels()
            self.root.after(400, self.ai_turn)  # небольшая паузка для живости

    def make_move(self, i):
        self.board[i] = self.current
        color = "#0044CC" if self.current == X else "#CC0000"
        self.buttons[i].config(text=self.current, fg=color)

        w, line = winner(self.board)
        if w:
            self.finish(w, line)
            return
        self.current = O if self.current == X else X
        self.update_labels()

    def ai_turn(self):
        if self.game_over:
            return
        p = LEVELS[self.ai_level]
        empties = [i for i in range(9) if self.board[i] == EMPTY]
        if random.random() > p:
            move = random.choice(empties)          # случайный ход (ошибка ИИ)
        else:
            move = best_move(self.board)           # лучший ход
        self.make_move(move)

    def finish(self, w, line):
        self.game_over = True
        self.score[w] += 1
        if line:
            for i in line:
                self.buttons[i].config(bg="#90EE90")
        if w == "draw":
            msg = "Ничья!"
        elif self.vs_ai:
            msg = "Ты победил! 🎉" if w == X else "ИИ победил. Попробуй ещё!"
        else:
            msg = f"Победил {w}! 🎉"
        self.status_var.set(msg)
        self.score_var.set(
            f"X: {self.score['X']}   O: {self.score['O']}   Ничьи: {self.score['draw']}"
        )


if __name__ == "__main__":
    root = tk.Tk()
    game = TicTacToe(root)
    root.mainloop()
