import sqlite3
import os

DB_NAME = "guia_oleo_silva.db"

def inicializar_banco():
    """Cria a tabela, a view e insere os dados iniciais se ainda não existirem."""
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()

    # 1. Criação da tabela oficial
    cur.execute("""
    CREATE TABLE IF NOT EXISTS catalogo_lubrificantes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        categoria TEXT,
        marca TEXT,
        modelo TEXT,
        motorizacao TEXT,
        ano_inicio INTEGER,
        ano_fim INTEGER,
        viscosidade TEXT,
        norma_homologada TEXT,
        capacidade_com_filtro REAL,
        tipo_base TEXT,
        alerta_critico TEXT
    )
    """)

    # 2. View para manter compatibilidade com consultas usando 'veiculos_oleo'
    cur.execute("""
    CREATE VIEW IF NOT EXISTS veiculos_oleo AS
    SELECT 
        id,
        categoria,
        marca,
        modelo,
        motorizacao,
        ano_inicio,
        ano_fim,
        viscosidade,
        norma_homologada AS norma_fabricante,
        capacidade_com_filtro,
        tipo_base,
        alerta_critico AS aviso_tecnico
    FROM catalogo_lubrificantes
    """)

    # 3. Verifica se já existem dados
    cur.execute("SELECT COUNT(*) FROM catalogo_lubrificantes")
    total = cur.fetchone()[0]

    if total == 0:
        dados_iniciais = [
            (
                'Leve', 'Chevrolet', 'Onix / Tracker', '1.0 3 Cil. Flex / Turbo', 
                2020, 2026, '0W-20', 'Dexos 1 Gen 3 (ou Gen 2)', 3.75, '100% Sintético',
                'Correia banhada a óleo: óleo errado desfaz a borracha da correia e trava o motor.'
            ),
            (
                'Leve', 'Volkswagen', 'Gol / Voyage / Fox', '1.0 / 1.6 EA111', 
                2008, 2023, '5W-40', 'VW 502.00 / VW 505.00', 3.80, '100% Sintético',
                'Exige homologação VW 502.00 para evitar borra no cabeçote.'
            ),
            (
                'Leve', 'Volkswagen', 'Polo / T-Cross / Nivus', '1.0 TSI', 
                2018, 2026, '0W-20', 'VW 508.88 / VW 502.00', 3.80, '100% Sintético',
                'Motores modernos TSI exigem óleo 100% sintético de baixa fricção.'
            ),
            (
                'Leve', 'Fiat', 'Argo / Strada / Mobi', '1.0 / 1.3 Firefly', 
                2017, 2026, '0W-20', 'Fiat 9.55535-GS1 / ACEA C2', 3.40, '100% Sintético',
                'Selenia K Forward 0W-20 ou similar 100% sintético homologado.'
            ),
            (
                'Leve', 'Toyota', 'Corolla', '1.8 / 2.0 16V Dual VVT-i', 
                2015, 2026, '0W-20', 'API SP / ILSAC GF-6A', 4.20, '100% Sintético',
                'Alta fluidez na partida a frio para correta atuação do variador VVT-i.'
            ),
            (
                'Leve', 'Hyundai', 'HB20 / Creta', '1.0 / 1.6 16V Kappa', 
                2013, 2026, '5W-30', 'API SP / ILSAC GF-6', 3.30, '100% Sintético',
                'Sintético com aditivação antidesgaste para comando de válvulas variável.'
            ),
            (
                'Camionete', 'Toyota', 'Hilux / SW4', '2.8 Turbo Diesel', 
                2016, 2026, '5W-30', 'ACEA C2 / Low-SAPS', 7.50, '100% Sintético',
                'Filtro DPF: óleo convencional entope o filtro de partículas do escapamento.'
            ),
            (
                'Camionete', 'Ford', 'Ranger', '2.2 / 3.2 TDCi Diesel', 
                2013, 2023, '5W-30', 'Ford WSS-M2C913-D', 9.80, '100% Sintético',
                'Grande volume de cárter. Exige norma específica Ford Duratorq.'
            ),
            (
                'Camionete', 'Chevrolet', 'S10', '2.8 CTDI Turbo Diesel', 
                2014, 2026, '5W-30', 'Dexos 2 / ACEA C3', 5.70, '100% Sintético',
                'Atenção: Em motores diesel com DPF, a norma é Dexos 2 (nunca Dexos 1).'
            ),
            (
                'Camionete', 'Fiat', 'Toro / Compass', '2.0 Diesel', 
                2016, 2024, '0W-30 ou 5W-30', 'Fiat 9.55535-DS1 / ACEA C2', 4.80, '100% Sintético',
                'Lubrificante de baixo teor de cinzas sulfatadas (Low-SAPS).'
            ),
            (
                'Camionete', 'Nissan', 'Frontier', '2.3 Bi-Turbo Diesel', 
                2017, 2026, '5W-30', 'ACEA C4 ou C3 / Low-SAPS', 6.80, '100% Sintético',
                'Exige óleo Low-SAPS para proteção do sistema bi-turbo e filtro DPF.'
            ),
        ]

        cur.executemany("""
        INSERT INTO catalogo_lubrificantes (
            categoria, marca, modelo, motorizacao, ano_inicio, ano_fim,
            viscosidade, norma_homologada, capacidade_com_filtro, tipo_base, alerta_critico
        ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, dados_iniciais)
        conn.commit()

    conn.close()

def buscar_oleo(veiculo_digitado):
    """Busca por modelo, marca ou motorização no catálogo."""
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()

    query = """
    SELECT 
        marca, 
        modelo, 
        motorizacao, 
        ano_inicio, 
        ano_fim, 
        viscosidade, 
        norma_homologada, 
        capacidade_com_filtro, 
        tipo_base, 
        alerta_critico,
        categoria
    FROM catalogo_lubrificantes
    WHERE modelo LIKE ? OR marca LIKE ? OR motorizacao LIKE ?
    """
    termo = f"%{veiculo_digitado.strip()}%"
    cur.execute(query, (termo, termo, termo))
    resultados = cur.fetchall()
    conn.close()

    if not resultados:
        return f"\n❌ Nenhum veículo encontrado para: '{veiculo_digitado}'. Digite 'listar' para ver os disponíveis."

    resumo = []
    for r in resultados:
        card = f"""
============================================================
🚗 VEÍCULO: {r[0]} {r[1]} [{r[10]}]
⚙️ Motor: {r[2]} ({r[3]} a {r[4]})
💧 Viscosidade Recomendada: {r[5]} ({r[8]})
📋 Norma / Homologação: {r[6]}
🛢️ Capacidade com Filtro: {r[7]:.2f} Litros
⚠️ Alerta Técnico Crítico: {r[9]}
============================================================
"""
        resumo.append(card)

    return "\n".join(resumo)

def listar_todos():
    """Exibe todos os modelos cadastrados no banco."""
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()
    cur.execute("SELECT marca, modelo, motorizacao, viscosidade FROM catalogo_lubrificantes ORDER BY categoria, marca")
    linhas = cur.fetchall()
    conn.close()
    
    print("\n--- VEÍCULOS DISPONÍVEIS NO CATÁLOGO ---")
    for l in linhas:
        print(f"• {l[0]} {l[1]} ({l[2]}) -> {l[3]}")
    print("----------------------------------------\n")

if __name__ == "__main__":
    inicializar_banco()
    print("============================================================")
    print("      CENTRO AUTOMOTIVO SILVA - GUIA DE LUBRIFICANTES       ")
    print("============================================================")
    print("Banco de dados pronto!")
    print("Dica: Digite o nome do carro, 'listar' para ver todos ou 'sair'.")

    while True:
        carro = input("\nDigite o modelo do veículo (ex: Hilux, Onix, Toro) ou 'sair': ").strip()
        if not carro:
            continue
        if carro.lower() == 'sair':
            print("\nEncerrando o Guia de Lubrificantes. Até mais!")
            break
        if carro.lower() == 'listar':
            listar_todos()
            continue
            
        print(buscar_oleo(carro))
