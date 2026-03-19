import asyncio
import socket
import tkinter as tk
from tkinter import ttk

# Async port scan
async def scan_port(host, port, output):
    try:
        reader, writer = await asyncio.open_connection(host, port)
        output.insert(tk.END, f"Port {port} is OPEN\n")
        writer.close()
        await writer.wait_closed()
    except:
        pass

# Async scan range
async def scan_range(host, start, end, output, progress):
    tasks = []
    total = end - start + 1

    for i, port in enumerate(range(start, end + 1)):
        tasks.append(scan_port(host, port, output))
        progress['value'] = (i / total) * 100
        root.update_idletasks()

    await asyncio.gather(*tasks)
    progress['value'] = 100

# Run async loop inside Tkinter
def start_scan():
    host = entry_host.get()
    try:
        start_port = int(entry_start.get())
        end_port = int(entry_end.get())
    except:
        output.insert(tk.END, "Invalid port range\n")
        return

    output.delete(1.0, tk.END)
    progress['value'] = 0

    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    loop.run_until_complete(scan_range(host, start_port, end_port, output, progress))
    loop.close()

# GUI setup
root = tk.Tk()
root.title("Async Port Scanner")
root.geometry("400x350")

tk.Label(root, text="Host").pack()
entry_host = tk.Entry(root)
entry_host.pack()

tk.Label(root, text="Start Port").pack()
entry_start = tk.Entry(root)
entry_start.pack()

tk.Label(root, text="End Port").pack()
entry_end = tk.Entry(root)
entry_end.pack()

tk.Button(root, text="Start Scan", command=start_scan).pack(pady=5)

progress = ttk.Progressbar(root, length=300, mode='determinate')
progress.pack(pady=5)

output = tk.Text(root, height=12)
output.pack()

root.mainloop()
