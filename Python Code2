import os
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# 1. Makale_Analiz_Tablolari.xlsx dosyasını proje klasöründe otomatik bulma
ana_dizin = r"C:\Users\User\Desktop\Yusuf İş\Adiyaman_Gee_Project"
excel_yolu = None

for kok, dizinler, dosyalar in os.walk(ana_dizin):
    if "Makale_Analiz_Tablolari.xlsx" in dosyalar:
        excel_yolu = os.path.join(kok, "Makale_Analiz_Tablolari.xlsx")
        break

if excel_yolu is None:
    raise FileNotFoundError("Makale_Analiz_Tablolari.xlsx dosyası Adiyaman_Gee_Project içinde bulunamadı!")

klasor = os.path.dirname(excel_yolu)
print("Excel dosyası bulundu:", excel_yolu)

# 2. Verileri okuma
alan_ozet = pd.read_excel(excel_yolu, sheet_name="Tablo3_Sinif_Alanlari")
matris_22_23 = pd.read_excel(excel_yolu, sheet_name="Tablo4_Gecis_2022_2023").set_index("From_Class")
matris_22_26 = pd.read_excel(excel_yolu, sheet_name="Tablo5_Gecis_2022_2026").set_index("From_Class")
ilce_temiz = pd.read_excel(excel_yolu, sheet_name="Tablo6_Ilce_Indeksler")

# 3. Sınıf ve ilçe adlarını Belirleme
en_sinif_map = {
    "Su": "Water Bodies",
    "Orman/Bahce": "Forest & Orchards",
    "Tarim": "Cropland",
    "Mera/Bozkir": "Rangeland / Steppe",
    "Yapilasmis": "Built-up Area",
    "Ciplak/Enkaz": "Bare Soil / Debris"
}
en_siniflar = list(en_sinif_map.values())

ilce_temiz["ADM2_NAME"] = ilce_temiz["ADM2_NAME"].replace({
    "Celikhan": "Çelikhan",
    "Golbasi": "Gölbaşı"
})

matris_22_23 = matris_22_23.rename(index=en_sinif_map, columns=en_sinif_map)
matris_22_26 = matris_22_26.rename(index=en_sinif_map, columns=en_sinif_map)

sns.set_style("whitegrid")
w = 0.27

# FIGURE 3: District-Level NDVI & NDBI Comparison
fig, axes = plt.subplots(2, 1, figsize=(12, 9.5), sharex=True)
x = np.arange(len(ilce_temiz["ADM2_NAME"]))

axes[0].bar(x - w, ilce_temiz["NDVI_2022"], w, label="2022 (Pre-Earthquake)", color="#2b8cbe")
axes[0].bar(x, ilce_temiz["NDVI_2023"], w, label="2023 (Post-Earthquake Year 1)", color="#e34a33")
axes[0].bar(x + w, ilce_temiz["NDVI_2026"], w, label="2026 (Reconstruction Period)", color="#31a354")
axes[0].set_ylabel("Mean NDVI", fontsize=11, fontweight="bold")
axes[0].set_title("Pre- and Post-Earthquake Vegetation (NDVI) and Built-up (NDBI) Dynamics by District", fontsize=13, fontweight="bold")
axes[0].legend(frameon=True, fontsize=10)

axes[1].bar(x - w, ilce_temiz["NDBI_2022"], w, label="2022 (Pre-Earthquake)", color="#2b8cbe")
axes[1].bar(x, ilce_temiz["NDBI_2023"], w, label="2023 (Post-Earthquake Year 1)", color="#e34a33")
axes[1].bar(x + w, ilce_temiz["NDBI_2026"], w, label="2026 (Reconstruction Period)", color="#31a354")
axes[1].set_ylabel("Mean NDBI", fontsize=11, fontweight="bold")
axes[1].set_xticks(x)
axes[1].set_xticklabels(ilce_temiz["ADM2_NAME"], rotation=30, ha="right", fontsize=10.5, fontweight="bold")

plt.tight_layout()
plt.savefig(os.path.join(klasor, "Fig3_Ilce_NDVI_NDBI_Karsilastirma.png"), dpi=300)
plt.close()

# FIGURE 4: LULC Class Area Distribution (Hectares)
plt.figure(figsize=(11.5, 6))
x_s = np.arange(len(en_siniflar))
plt.bar(x_s - w, alan_ozet["2022_Hektar"], w, label="May–June 2022 (Pre-Earthquake)", color="#2b8cbe")
plt.bar(x_s, alan_ozet["2023_Hektar"], w, label="May–June 2023 (Post-Earthquake Year 1)", color="#e34a33")
plt.bar(x_s + w, alan_ozet["2026_Hektar"], w, label="May–June 2026 (Reconstruction Period)", color="#31a354")
plt.ylabel("Area (Hectares)", fontsize=11, fontweight="bold")
plt.title("Areal Distribution of Land Use/Land Cover (LULC) Classes in Adıyaman (2022–2026)", fontsize=12.5, fontweight="bold")
plt.xticks(x_s, en_siniflar, fontsize=10, fontweight="bold")
plt.legend(frameon=True, fontsize=10)
plt.tight_layout()
plt.savefig(os.path.join(klasor, "Fig4_LULC_Sinif_Alanlari.png"), dpi=300)
plt.close()

# FIGURE 5: LULC Transition Matrix Heatmaps
fig, axes = plt.subplots(1, 2, figsize=(16.5, 6.8))
sns.heatmap(matris_22_23, annot=True, fmt=".0f", cmap="YlOrRd", linewidths=0.5, ax=axes[0], cbar=False)
axes[0].set_title("(a) 2022 → 2023 Land-Cover Transition Matrix (ha)", fontsize=12, fontweight="bold")
axes[0].set_xlabel("2023 LULC Class", fontsize=10.5, fontweight="bold")
axes[0].set_ylabel("2022 LULC Class", fontsize=10.5, fontweight="bold")
axes[0].tick_params(axis="x", rotation=25)

sns.heatmap(matris_22_26, annot=True, fmt=".0f", cmap="YlOrRd", linewidths=0.5, ax=axes[1])
axes[1].set_title("(b) 2022 → 2026 Land-Cover Transition Matrix (ha)", fontsize=12, fontweight="bold")
axes[1].set_xlabel("2026 LULC Class", fontsize=10.5, fontweight="bold")
axes[1].set_ylabel("")
axes[1].tick_params(axis="x", rotation=25)

plt.tight_layout()
plt.savefig(os.path.join(klasor, "Fig5_LULC_Gecis_Matrisi_Heatmap.png"), dpi=300)
plt.close()

print("Tamamlandı! İngilizce 300 DPI grafikler şu klasöre kaydedildi:", klasor)
