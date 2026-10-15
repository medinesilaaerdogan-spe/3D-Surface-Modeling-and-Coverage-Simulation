# 3D-Surface-Modeling-and-Coverage-Simulation
Python simulation project for 3D surface modeling, tactical device positioning, and coverage analysis.
import numpy as np
import plotly.graph_objects as go

def _create_single_device_traces(kure_yaricapi=5, nokta_sayisi=75, nokta_uzunlugu=2.5, offset=(0, 0, 0)):
    """
    Belirli bir merkez koordinatta (offset) taktik cihazı ve
    etki/kapsama alanını temsil eden 3 boyutlu grafik izlerini (traces) oluşturur.
    """
    ox, oy, oz = offset
    traces = []

    # Cihazın merkez noktası
    traces.append(go.Scatter3d(
        x=[ox], y=[oy], z=[oz],
        mode='markers',
        marker=dict(size=kure_yaricapi, color=oz, colorscale='Viridis', opacity=0.9),
        name=f"Merkez Cihaz ({ox},{oy},{oz})"
    ))

    # Çevresel kapsama/nokta dağılımı simülasyonu (rastgele vektörel dağılım)
    theta = np.linspace(0, 2 * np.pi, nokta_sayisi)
    phi = np.linspace(0, np.pi, nokta_sayisi)

    x_vals = ox + nokta_uzunlugu * np.outer(np.cos(theta), np.sin(phi)).flatten()[:20]
    y_vals = oy + nokta_uzunlugu * np.outer(np.sin(theta), np.sin(phi)).flatten()[:20]
    z_vals = oz + nokta_uzunlugu * np.outer(np.ones(np.shape(theta)), np.cos(phi)).flatten()[:20]

    traces.append(go.Scatter3d(
        x=x_vals, y=y_vals, z=z_vals,
        mode='markers',
        marker=dict(size=3, color='orange', opacity=0.6),
        name="Kapsama Sinyalleri"
    ))

    return traces

def grid_nokta_konumlandirma_olustur(grid_boyutu=(3, 3, 3), mesafe=13, kure_yaricapi=5, nokta_sayisi=75, nokta_uzunlugu=2.5):
    """
    Arazi üzerinde grid yapısında çoklu cihaz yerleşimini ve 3B görselleştirmesini yapar.
    """
    fig = go.Figure()
    nx, ny, nz = grid_boyutu
    koordinatlar = []

    for i in range(nx):
        for j in range(ny):
            for k in range(nz):
                offset = (i * mesafe, j * mesafe, k * mesafe)
                koordinatlar.append(offset)

                # Tekil cihaz izlerini ana figüre ekle
                traces = _create_single_device_traces(
                    kure_yaricapi=kure_yaricapi,
                    nokta_sayisi=nokta_sayisi,
                    nokta_uzunlugu=nokta_uzunlugu,
                    offset=offset
                )
                for trace in traces:
                    fig.add_trace(trace)

    fig.update_layout(
        title="3D Taktik Saha Cihaz Konumlandırma ve Kapsama Grid Simülasyonu",
        scene=dict(
            xaxis_title="X Koordinatı (Metre)",
            yaxis_title="Y Koordinatı (Metre)",
            zaxis_title="Z Yükseklik",
            aspectmode='data'
        ),
        margin=dict(l=0, r=0, b=0, t=40)
    )
    fig.show()
    return koordinatlar

def kapsama_ve_verim_analizi_simulasyonu(koordinatlar, max_menzil=25.0):
    """
    Yerleştirilen cihazların birbirleriyle olan mesafelerini hesaplayarak
    sistem verimini (throughput) ve aktif bağlantı sayısını simüle eder.
    """
    n_cihaz = len(koordinatlar)
    print("--- SAYZEK Konu 28: Kapsama ve Verim Analizi ---")
    print(f"Toplam Konumlandırılan Cihaz Sayısı: {n_cihaz}")

    koordinat_arr = np.array(koordinatlar)
    toplam_verim = 0
    kapsanan_baglanti_sayisi = 0

    for i in range(n_cihaz):
        for j in range(i + 1, n_cihaz):
            # Öklid mesafesi üzerinden sinyal kaybı/menzil hesabı
            mesafe = np.linalg.norm(koordinat_arr[i] - koordinat_arr[j])
            if mesafe <= max_menzil:
                # Menzil içindeyse verim skoru hesapla
                verim = max(0, 100 - (mesafe / max_menzil) * 50)
                toplam_verim += verim
                kapsanan_baglanti_sayisi += 1

    print(f"Optimizasyon Sonucu Toplam Sistem Verimi: {toplam_verim:.2f}")
    print(f"Aktif Bağlantı Sayısı: {kapsanan_baglanti_sayisi}")
    return toplam_verim

# --- Simülasyonu Çalıştırma Bloğu ---
if __name__ == "__main__":
    # 2x2x2'lik grid yapısında cihazları konumlandır ve 3B görselleştir
    koordinat_listesi = grid_nokta_konumlandirma_olustur(grid_boyutu=(2, 2, 2), mesafe=15)

    # Cihazlar arası verim ve kapsama analizini çalıştır
    kapsama_ve_verim_analizi_simulasyonu(koordinat_listesi)
    # 3x3x3 boyutundaki grid yapısı için simülasyonu yeniden çalıştırma
if __name__ == "__main__":
    # 3x3x3'lük grid yapısında (toplam 27 cihaz) konumlandırma ve görselleştirme
    koordinat_listesi_3x3 = grid_nokta_konumlandirma_olustur(grid_boyutu=(3, 3, 3), mesafe=15)
    
    # 3x3x3 yerleşimi için verim ve kapsama analizini çalıştır
    kapsama_ve_verim_analizi_simulasyonu(koordinat_listesi_3x3)
