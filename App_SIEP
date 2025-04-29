import streamlit as st
import numpy as np
import pandas as pd
import plotly.graph_objects as go
from scipy.stats import binom
from PIL import Image  # opcional – remova se não usar logo

# ─── Configuração da página ───────────────────────────────────── #
st.set_page_config(
    page_title="Simulador Overbooking & ROI",
    layout="wide",
    initial_sidebar_state="expanded"
)

# ─── Cabeçalho (logo opcional) ─────────────────────────────────── #
col_logo, col_title, col_logo2 = st.columns([1, 6, 1])
with col_logo:
    try:
        st.image(Image.open("Logo/MARCADOR.png"), use_column_width=True)
    except FileNotFoundError:
        pass

with col_title:
    st.markdown(
        "<h1 style='text-align:center;color:#003366'>Overbooking ✈️  &  ROI 💻</h1>",
        unsafe_allow_html=True
    )
st.markdown("---")

# ─── Criação das abas ─────────────────────────────────────────── #
tabs = st.tabs(["Overbooking (Binomial)", "ROI do Sistema (Monte Carlo)"])

# ═════════════════ 1) Overbooking ═══════════════════════════════ #
with tabs[0]:
    st.header("Risco de Overbooking")

    p = st.slider("Probabilidade de comparecimento (%)", 70, 100, 88, 1) / 100
    capacidade = 120  # valor fixo da atividade
    vendidas = st.slider("Passagens vendidas", capacidade, 160, 130, 1)

    risco = 1 - binom.cdf(capacidade, vendidas, p)

    faixa = np.arange(capacidade + 1, vendidas + 1)
    y = 1 - binom.cdf(capacidade, faixa, p)

    fig_over = go.Figure()
    fig_over.add_trace(go.Scatter(
        x=faixa, y=y,
        mode="lines", line=dict(width=3, color="#003366")
    ))
    fig_over.add_hline(y=0.07, line_dash="dash", line_color="red")
    fig_over.update_layout(
        title="Probabilidade de Exceder 120 Lugares",
        xaxis_title="Passagens Vendidas",
        yaxis_title="Probabilidade",
        plot_bgcolor="white"
    )
    st.plotly_chart(fig_over, use_container_width=True)

    st.success(f"Risco atual: **{risco*100:.2f}%**")

    st.markdown("##### Tabela detalhada")
    st.dataframe(pd.DataFrame({
        "Passagens Vendidas": faixa,
        "Risco (%)": np.round(y * 100, 2)
    }), height=240)

# ═════════════════ 2) ROI do Sistema ════════════════════════════ #
with tabs[1]:
    st.header("Simulação Interativa de ROI")

    colA, colB = st.columns(2)
    with colA:
        investimento = st.number_input(
            "Investimento inicial (R$)", 10000, 300000, 50000, 5000
        )
        receita_esp = st.slider(
            "Receita adicional esperada (R$)", 40000, 200000, 80000, 5000
        )
        meta_roi = st.number_input(
            "Meta de ROI (%)", 0.0, 300.0, 50.0, 1.0, format="%.1f"
        )
    with colB:
        custo_op = st.number_input(
            "Custo operacional anual (R$)", 0, 100000, 10000, 2000
        )
        desvio_rec = st.slider(
            "Desvio-padrão da receita (R$)", 1000, 60000, 15000, 1000
        )
        n_sims = st.slider(
            "Nº de simulações Monte Carlo", 500, 20000, 5000, 500
        )

    # Simulação Monte Carlo
    rng = np.random.default_rng(42)
    receitas = rng.normal(receita_esp, desvio_rec, n_sims)
    lucro = receitas - custo_op
    roi = lucro / investimento * 100  # ROI em %

    # Histograma do ROI
    fig_roi = go.Figure()
    fig_roi.add_trace(go.Histogram(
        x=roi, nbinsx=40, marker_color="#003366", opacity=0.75
    ))
    fig_roi.add_vline(x=0, line_dash="dash", line_color="red")
    fig_roi.update_layout(
        title="Distribuição Simulada do ROI (%)",
        xaxis_title="ROI (%)",
        yaxis_title="Frequência",
        plot_bgcolor="white"
    )
    st.plotly_chart(fig_roi, use_container_width=True)

    prob_neg = (roi < 0).mean() * 100
    prob_meta = (roi >= meta_roi).mean() * 100

    m1, m2, m3, m4 = st.columns(4)
    m1.metric("ROI médio", f"{roi.mean():.2f}%")
    m2.metric("ROI mín.", f"{roi.min():.2f}%")
    m3.metric("ROI máx.", f"{roi.max():.2f}%")
    m4.metric("Prob. ROI < 0", f"{prob_neg:.2f}%")

    st.info(f"Probabilidade de ROI ≥ {meta_roi:.1f}%: **{prob_meta:.2f}%**")
