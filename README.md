import pandas as pd
import os

# Lista dos arquivos de vendas
arquivos = [
    "Meganium_Sales_Data_-_AliExpress.csv",
    "Meganium_Sales_Data_-_Etsy.csv",
    "Meganium_Sales_Data_-_Shopee.csv",
    "Updated_Anbernic_Sales_Data.csv"
]

# Função para gerar SKU fictício com base no nome do produto
def gerar_sku_ficticio(produto):
    base = ''.join(e for e in produto if e.isalnum()).upper()
    return f"SKU-{base[:8]}"

# Carregar e padronizar os arquivos
dataframes = []
for arquivo in arquivos:
    if os.path.exists(arquivo):
        df = pd.read_csv(arquivo)

        # Gerar SKU se estiver ausente (caso do Anbernic)
        if "SKU" not in df.columns:
            df["SKU"] = df["product_sold"].apply(gerar_sku_ficticio)

        # Selecionar e ordenar colunas compatíveis
        colunas_padrao = [
            "SKU", "product_sold", "date", "quantity", "unit_price", "total_price",
            "currency", "site", "discount_coupon", "discount_value",
            "buyer_birth_date", "buyer_name", "delivery_country", "invoice_id"
        ]
        df = df[colunas_padrao]
        dataframes.append(df)
    else:
        print(f"Arquivo não encontrado: {arquivo}")

# Consolida os dados
df_total = pd.concat(dataframes, ignore_index=True)

# Solicita o país ao usuário
pais = input("Digite o nome do país (ex: Germany, Canada, Japan): ").strip()

# Filtra pelo país
df_filtrado = df_total[df_total["delivery_country"].str.lower() == pais.lower()]

# Verifica se há vendas no país informado
if df_filtrado.empty:
    print(f"\n❌ Nenhuma venda encontrada para o país: {pais}")
else:
    produto_top = (
        df_filtrado.groupby("product_sold")["quantity"]
        .sum()
        .sort_values(ascending=False)
        .reset_index()
        .iloc[0]
    )
    print(f"\n✅ Produto mais vendido em {pais}:")
    print(f"- {produto_top['product_sold']} ({produto_top['quantity']} unidades)")
