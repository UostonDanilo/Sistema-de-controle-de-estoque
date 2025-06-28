# Sistema-de-controle-de-estoque
Sistema de controle de estoque com interface gráfica em Python

Codigo Fonte:

import tkinter as tk
from tkinter import messagebox
import sqlite3

# Interage ao banco de dados
try:
    conn = sqlite3.connect('estoque.db')
    cursor = conn.cursor()
except sqlite3.Error as e:
    messagebox.showerror("Erro", f"Falha ao conectar ao banco: {e}")

# Cria a tabela, se não existir
cursor.execute('''
    CREATE TABLE IF NOT EXISTS produtos (
        nome TEXT PRIMARY KEY,
        quantidade INTEGER,
        preco REAL
    )
''')
conn.commit()

# Funções do sistema
def adicionar_produto():
    nome = entry_nome.get().strip()
    quantidade = entry_quantidade.get().strip()
    preco = entry_preco.get().strip()

    if not nome or not quantidade or not preco:
        messagebox.showwarning("Atenção", "Preencha todos os campos!")
        return

    try:
        cursor.execute('INSERT OR REPLACE INTO produtos VALUES (?, ?, ?)', (nome, int(quantidade), float(preco)))
        conn.commit()
        listar_produtos()
        limpar_campos()
        messagebox.showinfo("Sucesso", f"Produto {nome} foi adicionado!")
    except sqlite3.Error as e:
        messagebox.showerror("Erro", f"Falha ao salvar no banco: {e}")

def remover_produto():
    nome = entry_nome.get().strip()

    if not nome:
        messagebox.showwarning("Atenção", "Digite o nome do produto para remover!")
        return

    try:
        cursor.execute('DELETE FROM produtos WHERE nome = ?', (nome,))
        conn.commit()
        if cursor.rowcount == 0:
            messagebox.showinfo("Erro", "Produto não encontrado.")
        else:
            listar_produtos()
            limpar_campos()
            messagebox.showinfo("Sucesso", f"Produto {nome} foi removido!")
    except sqlite3.Error as e:
        messagebox.showerror("Erro", f"Falha ao excluir: {e}")

def listar_produtos():
    listbox.delete(0, tk.END)
    cursor.execute('SELECT * FROM produtos')
    produtos = cursor.fetchall()

    if not produtos:
        listbox.insert(tk.END, "Nenhum produto no estoque.")
    else:
        for nome, quantidade, preco in produtos:
            listbox.insert(tk.END, f"{nome} - Qtde: {quantidade}, R$ {preco:.2f}")

def limpar_campos():
    entry_nome.delete(0, tk.END)
    entry_quantidade.delete(0, tk.END)
    entry_preco.delete(0, tk.END)

def on_listbox_select(event):
    selecao = listbox.curselection()
    if selecao:
        valor = listbox.get(selecao[0])
        if "Nenhum produto" in valor:
            return  # Evita erro na mensagem do estoque vazio
        nome = valor.split(" - ")[0]
        cursor.execute('SELECT * FROM produtos WHERE nome = ?', (nome,))
        produto = cursor.fetchone()
        if produto:
            entry_nome.delete(0, tk.END)
            entry_nome.insert(0, produto[0])
            entry_quantidade.delete(0, tk.END)
            entry_quantidade.insert(0, produto[1])
            entry_preco.delete(0, tk.END)
            entry_preco.insert(0, produto[2])

# Interface
root = tk.Tk()
root.title("Controle de Estoque")

tk.Label(root, text="Nome:").grid(row=0, column=0)
entry_nome = tk.Entry(root)
entry_nome.grid(row=0, column=1)

tk.Label(root, text="Quantidade:").grid(row=1, column=0)
entry_quantidade = tk.Entry(root)
entry_quantidade.grid(row=1, column=1)

tk.Label(root, text="Preço:").grid(row=2, column=0)
entry_preco = tk.Entry(root)
entry_preco.grid(row=2, column=1)

tk.Button(root, text="Adicionar", command=adicionar_produto).grid(row=3, column=0)
tk.Button(root, text="Remover", command=remover_produto).grid(row=3, column=1)
tk.Button(root, text="Limpar", command=limpar_campos).grid(row=3, column=2)

listbox = tk.Listbox(root, width=50)
listbox.grid(row=4, column=0, columnspan=3)
listbox.bind("<<ListboxSelect>>", on_listbox_select)

listar_produtos()

root.mainloop()

# Fecha o banco apos sair do app
conn.close()
