import streamlit as st
import pandas as pd
import re

# Web sitesi başlığı
st.set_page_config(page_title="WhatsApp Analizcisi", page_icon="🕵️‍♂️")
st.title("WhatsApp Sohbet Analizcisi 🕵️‍♂️")

# Dosya yükleme alanı
uploaded_file = st.file_uploader("WhatsApp sohbet (.txt) dosyanızı yükleyin", type="txt")

if uploaded_file is not None:
    # Dosyayı okuma
    content = uploaded_file.getvalue().decode("utf-8")
    lines = content.split('\n')

    # WhatsApp txt dosyasından tarih, saat, gönderen ve mesajı ayıklamak için kural (Regex)
    # Örnek format: 12.05.2023 14:30 - Ali: Merhaba
    pattern = r"(\d{1,2}\.\d{1,2}\.\d{2,4}),?\s(\d{1,2}:\d{2})\s?-\s([^:]+):\s(.*)"
    data = []

    for line in lines:
        match = re.search(pattern, line)
        if match:
            tarih, saat, gonderen, mesaj = match.groups()
            data.append([f"{tarih} {saat}", gonderen, mesaj])

    if data:
        # Verileri bir tabloya (DataFrame) dönüştürme
        df = pd.DataFrame(data, columns=['TarihSaat', 'Gönderen', 'Mesaj'])
        df['TarihSaat'] = pd.to_datetime(df['TarihSaat'], format="%d.%m.%Y %H:%M", errors='coerce')
        
        st.success("Sohbet başarıyla yüklendi ve çözümlendi!")

        st.markdown("---")
        
        ### 1. İSTEK: KELİME ANALİZİ
        st.subheader("1. Kelime Kullanım Analizi")
        aranan_kelime = st.text_input("Kimin en çok kullandığını merak ettiğin kelimeyi yaz (örn: günaydın):").lower()
        
        if aranan_kelime:
            # Mesajları küçük harfe çevirip kelimeyi saydırıyoruz
            df['Kelime_Sayisi'] = df['Mesaj'].apply(lambda x: str(x).lower().count(aranan_kelime))
            kelime_toplam = df.groupby('Gönderen')['Kelime_Sayisi'].sum().reset_index()
            kelime_toplam = kelime_toplam.sort_values(by='Kelime_Sayisi', ascending=False)
            
            st.write(f"**'{aranan_kelime}'** kelimesinin kullanım sayıları:")
            st.dataframe(kelime_toplam, use_container_width=True)

        st.markdown("---")

        ### 2. İSTEK: YANIT SÜRESİ ANALİZİ
        st.subheader("2. Yanıt Süresi Analizi (Kim geç bakmış?)")
        if st.button("Ortalama Yanıt Sürelerini Hesapla"):
            # Zamana göre sırala
            df = df.sort_values('TarihSaat')
            
            # Bir önceki mesajla arasındaki zaman farkını hesapla
            df['Zaman_Farki'] = df['TarihSaat'].diff().dt.total_seconds()
            
            # Peş peşe atılan mesajları değil, sadece karşı tarafın ilk yanıtını baz almak için
            df['Gonderen_Degisti'] = df['Gönderen'] != df['Gönderen'].shift(1)
            
            # Sadece gönderen değiştiğinde geçen süreyi hesapla
            yanit_sureleri = df[df['Gonderen_Degisti']].groupby('Gönderen')['Zaman_Farki'].mean().reset_index()
            yanit_sureleri['Ortalama Yanıt (Dakika)'] = round(yanit_sureleri['Zaman_Farki'] / 60, 1)
            
            st.write("**Kişilerin bir mesaja ortalama cevap verme süreleri:**")
            st.dataframe(yanit_sureleri[['Gönderen', 'Ortalama Yanıt (Dakika)']], use_container_width=True)

    else:
        st.error("Dosya formatı anlaşılamadı. WhatsApp'tan dışa aktarılan orijinal txt dosyası olduğundan emin ol.")
