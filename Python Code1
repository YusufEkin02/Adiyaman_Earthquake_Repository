import ast
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats

#  GEE CIKTILARINI OKUMA
ilce_df = pd.read_csv("C:/Users/User/Desktop/Yusuf İş/Adiyaman_Gee_Project\gee_ciktilar/Ilce_Indeks_Istatistikleri.csv")
gecis_22_23 = pd.read_csv("C:/Users/User/Desktop/Yusuf İş/Adiyaman_Gee_Project\gee_ciktilar/LULC_Gecis_2022_2023.csv")
gecis_22_26 = pd.read_csv("C:/Users/User/Desktop/Yusuf İş/Adiyaman_Gee_Project\gee_ciktilar/LULC_Gecis_2022_2026.csv")
dogruluk_df = pd.read_csv("C:/Users/User/Desktop/Yusuf İş/Adiyaman_Gee_Project\gee_ciktilar/Dogruluk_Matrisi_2022.csv")

sinif_isimleri = ["Su", "Orman/Bahce", "Tarim", "Mera/Bozkir", "Yapilasmis", "Ciplak/Enkaz"]
sinif_map = {i: ad for i, ad in enumerate(sinif_isimleri)}

#  DOGRULUK MATRISINI (TABLO 2) DUZENLEME
oa = float(dogruluk_df["Overall_Accuracy"].iloc[0])
kappa = float(dogruluk_df["Kappa"].iloc[0])
cm_array = np.array(ast.literal_eval(dogruluk_df["Matrix"].iloc[0]))
prod_acc = np.array(ast.literal_eval(dogruluk_df["Producers_Accuracy"].iloc[0])).flatten() * 100
cons_acc = np.array(ast.literal_eval(dogruluk_df["Consumers_Accuracy"].iloc[0])).flatten() * 100

cm_tablo = pd.DataFrame(cm_array, index=sinif_isimleri, columns=sinif_isimleri)
cm_tablo["Producer_Accuracy (%)"] = np.round(prod_acc, 2)
cm_tablo["User_Accuracy (%)"] = np.round(cons_acc, 2)

print("=" * 60)
print(f"GENEL DOGRULUK (OA): %{oa * 100:.2f} | KAPPA: {kappa:.4f}")
print("=" * 60)
print("\n--- HATA MATRISI (CONFUSION MATRIX) ---")
print(cm_tablo)

# ARAZI ORTUSU GECIS MATRISLERI VE SINIF ALANLARI (HEKTAR)
def gecis_coz(df):
    df["From_Code"] = df["Transition_Code"] // 10
    df["To_Code"] = df["Transition_Code"] % 10
    df["From_Class"] = df["From_Code"].map(sinif_map)
    df["To_Class"] = df["To_Code"].map(sinif_map)
    matris = df.pivot_table(
        index="From_Class",
        columns="To_Class",
        values="Area_Hectare",
        aggfunc="sum",
        fill_value=0
    ).reindex(index=sinif_isimleri, columns=sinif_isimleri, fill_value=0)
    return df, matris

gecis_22_23, matris_22_23 = gecis_coz(gecis_22_23)
gecis_22_26, matris_22_26 = gecis_coz(gecis_22_26)

# Yillara gore toplam sinif alanlari (Hektar ve %)
alan_2022 = matris_22_23.sum(axis=1)
alan_2023 = matris_22_23.sum(axis=0)
alan_2026 = matris_22_26.sum(axis=0)

alan_ozet = pd.DataFrame({
    "Sinif": sinif_isimleri,
    "2022_Hektar": np.round(alan_2022.values, 2),
    "2022_Yuzde": np.round((alan_2022.values / alan_2022.sum()) * 100, 2),
    "2023_Hektar": np.round(alan_2023.values, 2),
    "2023_Yuzde": np.round((alan_2023.values / alan_2023.sum()) * 100, 2),
    "2026_Hektar": np.round(alan_2026.values, 2),
    "2026_Yuzde": np.round((alan_2026.values / alan_2026.sum()) * 100, 2),
    "Degisim_22_23_Ha": np.round(alan_2023.values - alan_2022.values, 2),
    "Degisim_22_26_Ha": np.round(alan_2026.values - alan_2022.values, 2)
})

print("\n--- YILLARA GORE ARAZI ORTUSU ALANLARI (HEKTAR) ---")
print(alan_ozet.to_string(index=False))

# ILCE BAZLI ISTATISTIKSEL ANLAMLILIK TESTLERI (PAIRED T-TEST & WILCOXON)
ilce_temiz = ilce_df.drop(columns=["system:index", ".geo"], errors="ignore").sort_values("ADM2_NAME")

test_sonuclari = []
for indeks in ["NDVI", "SAVI", "NDBI", "BSI"]:
    for yil_son in ["2023", "2026"]:
        s1 = ilce_temiz[f"{indeks}_2022"]
        s2 = ilce_temiz[f"{indeks}_{yil_son}"]
        t_stat, p_t = stats.ttest_rel(s1, s2)
        w_stat, p_w = stats.wilcoxon(s1, s2)
        test_sonuclari.append({
            "Indeks": indeks,
            "Karsilastirma": f"2022 vs {yil_son}",
            "Ort_2022": round(s1.mean(), 4),
            f"Ort_{yil_son}": round(s2.mean(), 4),
            "Fark": round(s2.mean() - s1.mean(), 4),
            "t_istatistigi": round(t_stat, 4),
            "Paired_t_p_degeri": round(p_t, 5),
            "Wilcoxon_p_degeri": round(p_w, 5)
        })

test_df = pd.DataFrame(test_sonuclari)
print("\n--- ISTATISTIKSEL HIPOTEZ TESTLERI (p < 0.05 ANLAMLI) ---")
print(test_df.to_string(index=False))

# TUM MAKALE TABLOLARINI TEK EXCEL DOSYASINA KAYDETME
with pd.ExcelWriter("C:/Users/User/Desktop/Yusuf İş/Adiyaman_Gee_Project/kod_sonuclari/Makale_Analiz_Tablolari.xlsx") as writer:
    cm_tablo.to_excel(writer, sheet_name="Tablo2_Dogruluk_Matrisi")
    alan_ozet.to_excel(writer, sheet_name="Tablo3_Sinif_Alanlari", index=False)
    np.round(matris_22_23, 2).to_excel(writer, sheet_name="Tablo4_Gecis_2022_2023")
    np.round(matris_22_26, 2).to_excel(writer, sheet_name="Tablo5_Gecis_2022_2026")
    ilce_temiz.to_excel(writer, sheet_name="Tablo6_Ilce_Indeksler", index=False)
    test_df.to_excel(writer, sheet_name="Tablo7_Istatistik_Testler", index=False)

#  MAKALE GRAFIKLERI (300 DPI)
sns.set_style("whitegrid")

# GRAFIK 1: Ilce Bazli NDVI ve NDBI Karsilastirmasi
fig, axes = plt.subplots(2, 1, figsize=(12, 10), sharex=True)
x = np.arange(len(ilce_temiz["ADM2_NAME"]))
w = 0.27

axes[0].bar(x - w, ilce_temiz["NDVI_2022"], w, label="2022 (Deprem Öncesi)", color="#2b8cbe")
axes[0].bar(x, ilce_temiz["NDVI_2023"], w, label="2023 (Deprem Sonrası 1. Yıl)", color="#e34a33")
axes[0].bar(x + w, ilce_temiz["NDVI_2026"], w, label="2026 (Yeniden İnşa Dönemi)", color="#31a354")
axes[0].set_ylabel("Ortalama NDVI", fontsize=11, fontweight="bold")
axes[0].set_title("Adıyaman İlçelerinde Deprem Öncesi ve Sonrası Bitki Örtüsü (NDVI) ve Yapılaşma (NDBI) Değişimi", fontsize=13, fontweight="bold")
axes[0].legend(frameon=True, fontsize=10)

axes[1].bar(x - w, ilce_temiz["NDBI_2022"], w, label="2022 (Deprem Öncesi)", color="#2b8cbe")
axes[1].bar(x, ilce_temiz["NDBI_2023"], w, label="2023 (Deprem Sonrası 1. Yıl)", color="#e34a33")
axes[1].bar(x + w, ilce_temiz["NDBI_2026"], w, label="2026 (Yeniden İnşa Dönemi)", color="#31a354")
axes[1].set_ylabel("Ortalama NDBI", fontsize=11, fontweight="bold")
axes[1].set_xticks(x)
axes[1].set_xticklabels(ilce_temiz["ADM2_NAME"], rotation=30, ha="right", fontsize=10, fontweight="bold")

plt.tight_layout()
plt.savefig("C:/Users/User/Desktop/Yusuf İş/Adiyaman_Gee_Project/kod_sonuclari/Fig3_Ilce_NDVI_NDBI_Karsilastirma.png", dpi=300)
plt.close()

# GRAFIK 2: Arazi Ortusu Sinif Alanlari (Hektar)
plt.figure(figsize=(11, 6))
x_s = np.arange(len(sinif_isimleri))
plt.bar(x_s - w, alan_ozet["2022_Hektar"], w, label="2022", color="#2b8cbe")
plt.bar(x_s, alan_ozet["2023_Hektar"], w, label="2023", color="#e34a33")
plt.bar(x_s + w, alan_ozet["2026_Hektar"], w, label="2026", color="#31a354")
plt.ylabel("Alan (Hektar)", fontsize=11, fontweight="bold")
plt.title("Adıyaman Arazi Örtüsü Sınıflarının Yıllara Göre Alansal Dağılımı (2022–2026)", fontsize=13, fontweight="bold")
plt.xticks(x_s, sinif_isimleri, fontsize=10, fontweight="bold")
plt.legend(frameon=True, fontsize=10)
plt.tight_layout()
plt.savefig("C:/Users/User/Desktop/Yusuf İş/Adiyaman_Gee_Project/kod_sonuclari/Fig4_LULC_Sinif_Alanlari.png", dpi=300)
plt.close()

# GRAFIK 3: Arazi Ortusu Gecis Matrisi Isi Haritasi (Heatmap)
fig, axes = plt.subplots(1, 2, figsize=(16, 6.5))
sns.heatmap(matris_22_23, annot=True, fmt=".0f", cmap="YlOrRd", linewidths=0.5, ax=axes[0], cbar=False)
axes[0].set_title("2022 → 2023 Arazi Geçiş Matrisi (Hektar)", fontsize=12, fontweight="bold")
axes[0].set_xlabel("2023 Sınıfı", fontsize=10, fontweight="bold")
axes[0].set_ylabel("2022 Sınıfı", fontsize=10, fontweight="bold")

sns.heatmap(matris_22_26, annot=True, fmt=".0f", cmap="YlOrRd", linewidths=0.5, ax=axes[1])
axes[1].set_title("2022 → 2026 Arazi Geçiş Matrisi (Hektar)", fontsize=12, fontweight="bold")
axes[1].set_xlabel("2026 Sınıfı", fontsize=10, fontweight="bold")
axes[1].set_ylabel("")

plt.tight_layout()
plt.savefig("C:/Users/User/Desktop/Yusuf İş/Adiyaman_Gee_Project/kod_sonuclari/Fig5_LULC_Gecis_Matrisi_Heatmap.png", dpi=300)
plt.close()

print("\nISLEM TAMAMLANDI! Excel tablolari '04_Manuscript', 300 DPI grafikler '03_Article_Figures' klasorune kaydedildi.")
