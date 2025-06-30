import os
import csv
from collections import defaultdict
from instaloader import Instaloader, Post
from reportlab.lib.pagesizes import A4
from reportlab.pdfgen import canvas

# Nama file dan folder
CSV_FILE = 'instagram_links.csv'
OUTPUT_DIR = 'downloaded_images'
PDF_FILE = 'Pose_Pack.pdf'

# Buat folder gambar jika belum ada
os.makedirs(OUTPUT_DIR, exist_ok=True)

# Inisialisasi Instaloader
loader = Instaloader(download_comments=False, save_metadata=False, post_metadata_txt_pattern="")

# Baca data dari CSV
entries = []
with open(CSV_FILE, newline='', encoding='utf-8') as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:
        entries.append(row)

# Kelompokkan berdasarkan kategori
by_category = defaultdict(list)
for entry in entries:
    shortcode = entry['url'].strip('/').split('/')[-1]
    try:
        post = Post.from_shortcode(loader.context, shortcode)
        filepath = os.path.join(OUTPUT_DIR, f"{shortcode}.jpg")
        if not os.path.exists(filepath):
            loader.download_pic(filepath, post.url, post.date)
        entry['img_path'] = filepath
        by_category[entry['category']].append(entry)
    except Exception as e:
        print(f"Failed to process {entry['url']}: {e}")

# Buat PDF
c = canvas.Canvas(PDF_FILE, pagesize=A4)
width, height = A4

for category, items in by_category.items():
    c.setFont("Helvetica-Bold", 16)
    c.drawString(100, height - 50, f"Kategori: {category}")
    y = height - 100
    for entry in items:
        if y < 100:
            c.showPage()
            y = height - 100
        if entry.get('img_path'):
            c.drawImage(entry['img_path'], 50, y - 150, width=200, height=150, preserveAspectRatio=True)
        c.setFont("Helvetica", 10)
        c.drawString(300, y, f"User: @{entry['username']}")
        c.drawString(300, y - 15, f"Lokasi: {entry['location_name']}")
        y -= 180
    c.showPage()

# Simpan PDF
c.save()
print(f"PDF saved as {PDF_FILE}")
 pose-pack-template
