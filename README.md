CREATE TABLE catalogo_lubrificantes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    categoria TEXT,              -- 'Leve' ou 'Camionete'
    marca TEXT,                  -- 'Chevrolet', 'Toyota', etc.
    modelo TEXT,                 -- 'Hilux', 'Onix', etc.
    motorizacao TEXT,            -- '2.8 Turbo Diesel', '1.0 12V 3 Cil'
    ano_inicio INTEGER,
    ano_fim INTEGER,
    viscosidade TEXT,            -- '0W-20', '5W-30', etc.
    norma_homologada TEXT,       -- 'Dexos 1 Gen 3', 'ACEA C2', 'VW 508.88'
    capacidade_com_filtro REAL,  -- Litros (ex: 7.5)
    tipo_base TEXT,              -- '100% Sintético', 'Semissintético'
    alerta_critico TEXT          -- Alertas de DPF, correia banhada a óleo, etc.
);Tabela de Aplicações Rápidas (Veículos Mais Comuns e Camionetes)CategoriaVeículo / MotorAnosViscosidadeNorma ObrigatóriaLitros (c/ filtro)Ponto Crítico da AplicaçãoLeveOnix / Tracker (1.0 3 Cil. Flex / Turbo)2020 a 20260W-20Dexos 1 Gen 3 (ou Gen 2)3,75 LCorreia banhada a óleo: óleo errado desfaz a borracha da correia e trava o motor.LeveGol / Voyage / Fox (1.0 / 1.6 EA111)2008 a 20235W-40VW 502.00 / VW 505.003,80 LExige homologação VW 502.00 para evitar borra no cabeçote.LevePolo / T-Cross / Nivus (1.0 TSI)2018 a 20260W-20VW 508.88 / VW 502.003,80 LMotores modernos TSI exigem óleo 100% sintético de baixa fricção.LeveArgo / Strada / Mobi (1.0 / 1.3 Firefly)2017 a 20260W-20Fiat 9.55535-GS1 / ACEA C23,40 LSelenia K Forward 0W-20 ou similar 100% sintético homologado.LeveCorolla (1.8 / 2.0 16V Dual VVT-i)2015 a 20260W-20API SP / ILSAC GF-6A4,20 LAlta fluidez na partida a frio para correta atuação do variador VVT-i.LeveHB20 / Creta (1.0 / 1.6 16V Kappa)2013 a 20265W-30API SP / ILSAC GF-63,30 LSintético com aditivação antidesgaste para comando de válvulas variável.CamioneteToyota Hilux / SW4 (2.8 Turbo Diesel)2016 a 20265W-30ACEA C2 / Low-SAPS7,50 LFiltro DPF: óleo convencional entope o filtro de partículas do escapamento.CamioneteFord Ranger (2.2 / 3.2 TDCi Diesel)2013 a 20235W-30Ford WSS-M2C913-D9,80 L (3.2)Grande volume de cárter. Exige norma específica Ford Duratorq.CamioneteChevrolet S10 (2.8 CTDI Turbo Diesel)2014 a 20265W-30Dexos 2 / ACEA C35,70 LAtenção: Em motores diesel com DPF, a norma é Dexos 2 (nunca Dexos 1).CamioneteFiat Toro / Compass (2.0 Diesel)2016 a 20240W-30 ou 5W-30Fiat 9.55535-DS1 / ACEA C24,80 LLubrificante de baixo teor de cinzas sulfatadas (Low-SAPS).CamioneteNissan Frontier (2.3 Bi-Turbo Diesel)2017 a 20265W-30
import sqlite3

def buscar_oleo(veiculo_digitado):
    conn = sqlite3.connect("guia_oleo_silva.db")
    cur = conn.cursor()
    
    query = """
    SELECT marca, modelo, motorizacao, ano_inicio, ano_fim, 
           viscosidade, norma_fabricante, capacidade_com_filtro, tipo_base, aviso_tecnico
    FROM veiculos_oleo
    WHERE modelo LIKE ? OR marca LIKE ?
    """
    termo = f"%{veiculo_digitado}%"
    cur.execute(query, (termo, termo))
    resultados = cur.fetchall()
    conn.close()
    
    if not resultados:
        return f"\n❌ Nenhum veículo encontrado para: '{veiculo_digitado}'. Verifique a digitação."
    
    resumo = []
    for r in resultados:
        card = f"""
============================================================
🚗 VEÍCULO: {r[0]} {r[1]} - Motor: {r[2]} ({r[3]} a {r[4]})
------------------------------------------------------------
🛢️ Viscosidade Recomendada: {r[5]} ({r[8]})
📜 Norma / Homologação:   {r[6]}
📏 Capacidade com Filtro:  {r[7]} Litros
⚠️ Alerta Técnico:         {r[9]}
============================================================
"""
        resumo.append(card)
    
    return "\n".join(resumo)

# Teste interativo no terminal:
if __name__ == "__main__":
    while True:
        carro = input("\nDigite o modelo do veículo (ex: Hilux, Onix, Toro) ou 'sair': ")
        if carro.lower() == 'sair':
            break
        print(buscar_oleo(carro))
