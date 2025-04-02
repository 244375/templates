from flask import Flask, request, jsonify
import sqlite3

app = Flask(__name__)

def init_db():
    with sqlite3.connect("os.db") as conn:
        cursor = conn.cursor()
        cursor.execute('''CREATE TABLE IF NOT EXISTS ordens (
                            id INTEGER PRIMARY KEY AUTOINCREMENT,
                            cliente TEXT,
                            descricao TEXT,
                            status TEXT)''')
        conn.commit()

@app.route("/os", methods=["POST"])
def criar_os():
    dados = request.json
    with sqlite3.connect("os.db") as conn:
        cursor = conn.cursor()
        cursor.execute("INSERT INTO ordens (cliente, descricao, status) VALUES (?, ?, ?)",
                       (dados["cliente"], dados["descricao"], "Aberto"))
        conn.commit()
    return jsonify({"mensagem": "Ordem de serviço criada com sucesso!"})

@app.route("/os", methods=["GET"])
def listar_os():
    with sqlite3.connect("os.db") as conn:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM ordens")
        ordens = cursor.fetchall()
    return jsonify(ordens)

@app.route("/os/<int:os_id>", methods=["PUT"])
def atualizar_os(os_id):
    dados = request.json
    with sqlite3.connect("os.db") as conn:
        cursor = conn.cursor()
        cursor.execute("UPDATE ordens SET status=? WHERE id=?", (dados["status"], os_id))
        conn.commit()
    return jsonify({"mensagem": "Ordem de serviço atualizada!"})

@app.route("/os/<int:os_id>", methods=["DELETE"])
def excluir_os(os_id):
    with sqlite3.connect("os.db") as conn:
        cursor = conn.cursor()
        cursor.execute("DELETE FROM ordens WHERE id=?", (os_id,))
        conn.commit()
    return jsonify({"mensagem": "Ordem de serviço excluída!"})

if __name__ == "__main__":
    init_db()
    app.run(debug=True)
