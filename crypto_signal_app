import pandas as pd
import numpy as np
import talib
import tkinter as tk
from tkinter import ttk
from tkinter import filedialog  
import threading
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from binance.client import Client
from binance.enums import HistoricalKlinesType
from datetime import datetime
import matplotlib.dates as mdates
import matplotlib.dates as mdates

pd.set_option('display.max_rows', None)  

client = Client()

def get_all_symbols():
    exchange_info = client.get_exchange_info()
    symbols = [symbol['symbol'] for symbol in exchange_info['symbols'] 
               if symbol['status'] == 'TRADING' and symbol['symbol'].endswith('USDT')]
    return symbols


def get_price_change(symbol, lookback_period):
    if lookback_period == "1 days ago UTC":
        interval = '1h'
    elif lookback_period == "10 days ago UTC":
        interval = '1d'
    elif lookback_period == "1 hour ago UTC":
        interval = '1m'
    elif lookback_period == "1 minutes ago UTC":
        interval = '1s'
    elif lookback_period == "5 minutes ago UTC":
        interval = '1m' 
    elif lookback_period == "30 minutes ago UTC":
        interval = '1m'
    else:
        interval = '1m'
    try:
        
        klines = client.get_historical_klines(symbol, interval, lookback_period, klines_type=HistoricalKlinesType.SPOT)
        
        
        if len(klines) < 2:
            return None
        
        
        open_price = float(klines[0][1])  
        close_price = float(klines[-1][4])  
        
        
        price_change = ((close_price - open_price) / open_price) * 100
        return round(price_change, 2)
    except Exception as e:
        print(f"Error fetching price change for {symbol} (Interval: {interval}, Lookback: {lookback_period}): {e}")
        return None
    


def calculate_rsi(symbol, rsiinterval_period, period):
    if rsiinterval_period == "1d":  
        xtimeago = "50 days ago UTC"
    elif rsiinterval_period == "1m":
        xtimeago = "50 minutes ago UTC"
    elif rsiinterval_period == "15m":
        xtimeago = "12 hours ago UTC"
    elif rsiinterval_period == "5m":
        xtimeago = "4 hours ago UTC"
    elif rsiinterval_period == "1h":
        xtimeago = "2 days ago UTC"
    elif rsiinterval_period == "4h":
        xtimeago = "4 days ago UTC"
    else:
        xtimeago = "30 days ago UTC"
    try:
        klines = client.get_historical_klines(symbol, rsiinterval_period, xtimeago, klines_type=HistoricalKlinesType.SPOT)
        closes = [float(kline[4]) for kline in klines if kline[4] is not None]
        
        if len(closes) < period:
            return None
        
        np_closes = np.array(closes[-(period+1):], dtype=float)
        rsi_values = talib.RSI(np_closes, timeperiod=period)
        rsi = round(rsi_values[-1], 2) if not np.isnan(rsi_values[-1]) else None
        return rsi
    except Exception as e:
        return None


# Seçilen coinin detaylarını gösteren pencere
def show_coin_details(event,tree):
    selected_item = tree.selection()
    if selected_item:
        item = tree.item(selected_item)
        symbol = item['values'][0]
        
        details_window = tk.Toplevel()
        details_window.title(f"{symbol} Details")
        details_window.geometry("900x800")  # Pencereyi büyüt
        
        ttk.Label(details_window, text=f"Symbol: {symbol}", font=("Arial", 12, "bold")).pack(pady=10)
        
        try:
            ticker = client.get_ticker(symbol=symbol)
            last_price = float(ticker['lastPrice'])  # Son fiyat
            price_change_percent = float(ticker['priceChangePercent'])  # 24 saatlik değişim yüzdesi
            high_price = float(ticker['highPrice'])  # 24 saatlik en yüksek fiyat
            low_price = float(ticker['lowPrice'])  # 24 saatlik en düşük fiyat
            volume = float(ticker['volume'])  # 24 saatlik işlem hacmi (base asset cinsinden)
            quote_volume = float(ticker['quoteVolume'])  # 24 saatlik işlem hacmi (quote asset cinsinde
            
            ttk.Label(details_window, text=f"Last Price: {last_price:.2f} USDT").pack()
            ttk.Label(details_window, text=f"24h Change: {price_change_percent:.2f}%").pack()
            ttk.Label(details_window, text=f"24h High: {high_price:.2f} USDT").pack()
            ttk.Label(details_window, text=f"24h Low: {low_price:.2f} USDT").pack()
            ttk.Label(details_window, text=f"24h Volume (Base): {volume:.2f} {symbol}").pack()
            ttk.Label(details_window, text=f"24h Volume (Quote): {quote_volume:.2f} USDT").pack()
            
        except Exception as e:
            ttk.Label(details_window, text="Error fetching details").pack()
        
        # Grafik Alanı
        figure, ax = plt.subplots(figsize=(9, 4))  # Daha büyük bir grafik
        canvas = FigureCanvasTkAgg(figure, details_window)
        canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True, padx=2, pady=1)  # Grafik alanı genişlesin
        
        # Zaman aralığı seçimi için butonlar
        button_frame = ttk.Frame(details_window)
        button_frame.pack(pady=10)
        
        def plot_candlestick(interval):
            if interval == "1d":
                timeago = "30 day ago UTC"
            elif interval == "1h":
                timeago = "3 day ago UTC"
            elif interval == "5m":
                timeago = "1 day ago UTC"
            klines = client.get_historical_klines(symbol, interval, timeago, klines_type=HistoricalKlinesType.SPOT)
            if klines:
                timestamps = [datetime.utcfromtimestamp(x[0] / 1000) for x in klines]  # Unix timestamp'leri tarih formatına çevir
                closing_prices = [float(x[4]) for x in klines]
                
                ax.clear()
                ax.plot(timestamps, closing_prices, label=f"{symbol} ({interval})", marker='o', linestyle='-')
                ax.legend()
                
                # X ekseni tarihlerini gizle
                ax.xaxis.set_major_formatter(plt.NullFormatter())
                
                # Tooltip için etkileşimli gösterge
                tooltip = ax.annotate("", xy=(0,0), xytext=(-40, 20),
                                    textcoords='offset points', bbox=dict(boxstyle='round,pad=0.3', edgecolor='black', facecolor='lightgray'))
                tooltip.set_visible(False)
                
                def on_hover(event):
                    if event.xdata is not None and event.ydata is not None:
                        # En yakın noktayı bul
                        distances = [(abs(event.xdata - mdates.date2num(ts)), price) for ts, price in zip(timestamps, closing_prices)]
                        closest_point = min(distances, key=lambda x: x[0])  # En yakın noktayı bul

                        # Belirli bir mesafeden uzaktaysa gösterme
                        if closest_point[0] < 0.20:  # Hassasiyet ayarı (daha küçük değer daha hassas)
                            date_str = mdates.num2date(event.xdata).strftime('%Y-%m-%d %H:%M')
                            price_str = f"Price: {closest_point[1]:.6f} USDT"
                            tooltip.set_text(f"{date_str}\n{price_str}")
                            tooltip.xy = (event.xdata, event.ydata)
                            tooltip.set_visible(True)
                        else:
                            tooltip.set_visible(False)

                    canvas.draw()
                
                figure.canvas.mpl_connect("motion_notify_event", on_hover)  # Mouse hareketi ile göstergeyi etkinleştir
                
                canvas.draw()
        plot_candlestick("1d")
        ttk.Button(button_frame, text="1D", command=lambda: plot_candlestick("1d")).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="1H", command=lambda: plot_candlestick("1h")).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="5M", command=lambda: plot_candlestick("5m")).pack(side=tk.LEFT, padx=5)

def analyze_cryptos(overbought, oversold, period, price_change_threshold, progress_var, count_var,lookback_period,rsiinterval_period):
    symbols = get_all_symbols()[:5]
    results = []
    total_symbols = len(symbols)

    for i, symbol in enumerate(symbols):
        try:
            price_change = get_price_change(symbol, lookback_period)
            rsi = calculate_rsi(symbol, rsiinterval_period , period)
            
            if price_change is not None and rsi is not None:
                recommendation = 'Neutral'
                if rsi < oversold:
                    recommendation = 'Buy (Oversold)'
                elif rsi > overbought:
                    recommendation = 'Sell (Overbought)'
                elif price_change > price_change_threshold:
                    recommendation = 'Buy (High Momentum)'
                
                results.append({
                    'Symbol': symbol,
                    'Change (%)': price_change,
                    'RSI': rsi,
                    'Recommendation': recommendation
                })
            
            progress_var.set(int(((i + 1) / total_symbols) * 100))
            count_var.set(f"Processed Coins: {i + 1}/{total_symbols}")
        except Exception as e:
            pass
    
    df = pd.DataFrame(results)
    df = df.sort_values(by=['Change (%)'], ascending=False)
    return df


def export_to_excel():
    
    file_path = filedialog.asksaveasfilename(defaultextension=".xlsx", filetypes=[("Excel files", "*.xlsx")])
    if file_path:
        
        results_df.to_excel(file_path, index=False)
        print(f"Data exported to {file_path}")

import tkinter.messagebox

def binance_weight():
    try:
        headers = client.response.headers
        used_weight = headers.get("X-MBX-USED-WEIGHT-1M", "Unknown")
        return f"Binance api weight used until the last minute: {used_weight}"
    except Exception as e:
        return f"Error: {e}"


def run_gui():
    global results_df  
    def fetch_and_display():
        def task():
            global results_df
            overbought = int(overbought_entry.get())
            oversold = int(oversold_entry.get())
            period = int(period_entry.get())
            price_change_threshold = float(price_change_entry.get())
            lookback_period = lookback_period_var.get()
            rsiinterval_period = rsiinterval_period_var.get()

            results_df = analyze_cryptos(overbought, oversold, period, price_change_threshold, progress_var, count_var,lookback_period,rsiinterval_period)
            tree.delete(*tree.get_children())
            filtered_tree.delete(*filtered_tree.get_children())
            
            for index, row in results_df.iterrows():
                tree.insert("", "end", values=(row['Symbol'], row['Change (%)'], row['RSI'], row['Recommendation']))
                if row['Recommendation'] != 'Neutral':
                    filtered_tree.insert("", "end", values=(row['Symbol'], row['Change (%)'], row['RSI'], row['Recommendation']))
            
            progress_var.set(100)
        
        threading.Thread(target=task, daemon=True).start()
    
    root = tk.Tk()
    root.title("Crypto Analyzer")
    
    
    # Menü çubuğunu oluştur
    menubar = tk.Menu(root)
    root.config(menu=menubar)
    
   
    # "File" menüsü ekle
    file_menu = tk.Menu(menubar, tearoff=0)
    menubar.add_cascade(label="File", menu=file_menu)
    file_menu.add_command(label="Export to Excel", command=export_to_excel)
    file_menu.add_separator()
    file_menu.add_command(label="Exit", command=root.quit)
    
    # "Help" menüsü ekle
    help_menu = tk.Menu(menubar, tearoff=0)
    menubar.add_cascade(label="Help", menu=help_menu)
    help_menu.add_command(label="Refresh", command=fetch_and_display)
    help_menu.add_command(label="About", command=lambda: tk.messagebox.showinfo("About", "Crypto Analyzer v1.0"))
    help_menu.add_command(label="Info", command=lambda: tk.messagebox.showinfo("Info", "Real-time data is being retrieved from Binance."))
    help_menu.add_command(label="Binance weight", command=lambda: tk.messagebox.showinfo("Binance weight", binance_weight()))


    price_change_frame = ttk.LabelFrame(root, text="Price Change Options")
    price_change_frame.grid(row=0, column=0, padx=10, pady=10, sticky="nw")
    
    ttk.Label(price_change_frame, text="Price Change Threshold:").grid(row=0, column=0)
    price_change_entry = ttk.Entry(price_change_frame)
    price_change_entry.grid(row=0, column=1)
    price_change_entry.insert(0, "5")
    
    ttk.Label(price_change_frame, text="Price Change Interval:").grid(row=1, column=0, padx=5, pady=5)
    lookback_period_var = ttk.Combobox(price_change_frame, values=["1 days ago UTC", "10 days ago UTC", "1 hour ago UTC", "1 minutes ago UTC", "5 minutes ago UTC", "30 minutes ago UTC"], state="readonly")
    lookback_period_var.grid(row=1, column=1, padx=5, pady=5)
    lookback_period_var.set("1 days ago UTC")
    
    rsi_frame = ttk.LabelFrame(root, text="RSI Options")
    rsi_frame.grid(row=0, column=1, padx=10, pady=10, sticky="ne")
    
    ttk.Label(rsi_frame, text="Overbought RSI:").grid(row=0, column=0)
    overbought_entry = ttk.Entry(rsi_frame)
    overbought_entry.grid(row=0, column=1)
    overbought_entry.insert(0, "70")
    
    ttk.Label(rsi_frame, text="Oversold RSI:").grid(row=1, column=0)
    oversold_entry = ttk.Entry(rsi_frame)
    oversold_entry.grid(row=1, column=1)
    oversold_entry.insert(0, "30")
    
    ttk.Label(rsi_frame, text="RSI Period:").grid(row=2, column=0)
    period_entry = ttk.Entry(rsi_frame)
    period_entry.grid(row=2, column=1)
    period_entry.insert(0, "14")

    ttk.Label(rsi_frame, text="Rsi Interval:").grid(row=3, column=0, padx=5, pady=5)
    rsiinterval_period_var = ttk.Combobox(rsi_frame, values=["1d", "1m", "15m", "5m", "1h", "4h"], state="readonly")
    rsiinterval_period_var.grid(row=3, column=1, padx=5, pady=5)
    rsiinterval_period_var.set("1d")
    
    analyze_button = ttk.Button(root, text="Analyze", command=fetch_and_display)
    analyze_button.grid(row=1, column=0, columnspan=4, pady=10)
    
    progress_var = tk.IntVar()
    count_var = tk.StringVar()
    count_var.set("Processed Coins: 0")
    
    progress_bar = ttk.Progressbar(root, orient="horizontal", length=300, mode="determinate", variable=progress_var)
    progress_bar.grid(row=3, column=0, columnspan=4, pady=5)
    
    count_label = ttk.Label(root, textvariable=count_var)
    count_label.grid(row=4, column=0, columnspan=4)

    recommendations_label = ttk.Label(root, text="All of Them", font=("Arial", 10, "bold"))
    recommendations_label.grid(row=7, column=0, columnspan=4, pady=5)
    
    frame = ttk.Frame(root)
    frame.grid(row=8, column=0, columnspan=4)
    
    
    tree_scroll = ttk.Scrollbar(frame, orient="vertical")
    tree_scroll.pack(side="right", fill="y")
    
    tree = ttk.Treeview(frame, columns=("Symbol", "Change (%)", "RSI", "Recommendation"), show='headings', yscrollcommand=tree_scroll.set)
    tree.pack()
    tree.bind("<Double-1>", lambda event: show_coin_details(event, tree))
    tree_scroll.config(command=tree.yview)

    recommendations_label = ttk.Label(root, text="Just Recommendations", font=("Arial", 10, "bold"))
    recommendations_label.grid(row=9, column=0, columnspan=4, pady=5)
    
    filtered_frame = ttk.Frame(root)
    filtered_frame.grid(row=10, column=0, columnspan=4)
    
    filtered_scroll = ttk.Scrollbar(filtered_frame, orient="vertical")
    filtered_scroll.pack(side="right", fill="y")
    
    filtered_tree = ttk.Treeview(filtered_frame, columns=("Symbol", "Change (%)", "RSI", "Recommendation"), show='headings', yscrollcommand=filtered_scroll.set)
    filtered_tree.pack()
    filtered_tree.bind("<Double-1>", lambda event: show_coin_details(event, filtered_tree))
    filtered_scroll.config(command=filtered_tree.yview)
    
    for col in ("Symbol", "Change (%)", "RSI", "Recommendation"):
        tree.heading(col, text=col)
        filtered_tree.heading(col, text=col)
    
    root.mainloop()

if __name__ == "__main__":
    run_gui()
