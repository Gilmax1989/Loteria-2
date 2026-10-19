# Lector de resultados de loterías dominicanas para GitHub Actions.
# Basado en la idea de https://github.com/CarlsRemy/LotteryScraping-RD (GPL v3).
# Lee loteriasdominicanas.com y escribe resultados.txt con líneas: fecha,sorteo,1ra,2da,3ra
import sys, re, time, datetime, urllib.request
from bs4 import BeautifulSoup

DIAS = 4  # cuántos días hacia atrás revisa en cada corrida
PAGINAS = ["https://loteriasdominicanas.com/", "https://loteriasdominicanas.com/anguila"]

# nombre en la página (minúsculas) -> nombre en tu página
NOMBRES = {
    "la primera día": "La Primera Tarde", "la primera dia": "La Primera Tarde",
    "primera noche": "La Primera Noche",
    "anguila mañana": "Anguila 10:00 AM", "anguila manana": "Anguila 10:00 AM",
    "anguila medio día": "Anguila 1:00 PM", "anguila medio dia": "Anguila 1:00 PM",
    "anguila tarde": "Anguila 6:00 PM",
    "anguila noche": "Anguila 9:00 PM",
    "la suerte 12:30": "La Suerte",
    "la suerte 18:00": "La Suerte 6PM",
    "quiniela real": "La Real",
    "florida día": "La Florida", "florida dia": "La Florida",
    "florida noche": "Florida Noche",
    "quiniela lotedom": "LoteDom",
    "new york tarde": "New York Tarde",
    "new york noche": "New York Noche",
    "gana más": "Nacional Tarde (Gana Más)", "gana mas": "Nacional Tarde (Gana Más)",
    "lotería nacional": "Nacional Noche", "loteria nacional": "Nacional Noche",
    "quiniela loteka": "Loteka",
    "quiniela leidsa": "Leidsa",
}

def bajar(url):
    req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0 (compatible; lector-personal)"})
    return urllib.request.urlopen(req, timeout=30).read()

def extraer(html, fecha):
    """Devuelve líneas (fecha, sorteo, n1, n2, n3) de un HTML."""
    out = []
    soup = BeautifulSoup(html, "html.parser")
    for b in soup.find_all("div", class_="game-block"):
        t = b.find("a", class_="game-title")
        if not t:
            continue
        nombre = NOMBRES.get(t.get_text().strip().lower())
        if not nombre:
            continue
        nums = [s.get_text().strip() for s in b.find_all("span", class_="score")]
        nums = [n.zfill(2) for n in nums if re.fullmatch(r"\d{1,2}", n)][:3]
        if len(nums) < 3:
            continue
        out.append((fecha, nombre, *nums))
    return out

def main():
    hoy = (datetime.datetime.now(datetime.timezone.utc) - datetime.timedelta(hours=4)).date()  # hora de RD
    nuevos = {}
    errores = 0
    for d in range(DIAS):
        dia = hoy - datetime.timedelta(days=d)
        q = dia.strftime("%d-%m-%Y")
        for base in PAGINAS:
            try:
                html = bajar(f"{base}?date={q}")
            except Exception as e:
                errores += 1
                print("No pude leer", base, q, e, file=sys.stderr)
                continue
            filas = extraer(html, dia.isoformat())
            sopa = BeautifulSoup(html, "html.parser")
            print("DIAG", base, q, "bytes:", len(html), "game-block:", len(sopa.find_all("div", class_="game-block")),
                  "título:", (sopa.title.get_text().strip() if sopa.title else "-"), "leídos:", len(filas))
            if not filas and d == 0:
                txt = re.sub(r"\s+", " ", sopa.get_text(" "))[:500]
                print("DIAG texto:", txt)
                print("DIAG clases:", sorted({c for t in sopa.find_all(True) for c in (t.get("class") or [])})[:80])
            for f, s, a, b, c in filas:
                nuevos[(f, s)] = (a, b, c)
            time.sleep(1)
    if not nuevos and errores:
        print("No se leyó nada; no se cambia el archivo.", file=sys.stderr)
        sys.exit(0)
    # une con lo que ya estaba (conserva días anteriores, los nuevos mandan)
    viejos = {}
    try:
        for l in open("resultados.txt", encoding="utf-8"):
            p = l.strip().split(",")
            if len(p) == 5:
                viejos[(p[0], p[1])] = tuple(p[2:])
    except FileNotFoundError:
        pass
    viejos.update(nuevos)
    corte = (hoy - datetime.timedelta(days=30)).isoformat()
    filas = sorted(((f, s) + n for (f, s), n in viejos.items() if f >= corte), reverse=True)
    with open("resultados.txt", "w", encoding="utf-8") as fh:
        for r in filas:
            fh.write(",".join(r) + "\n")
    print(len(nuevos), "resultados leídos;", len(filas), "en el archivo.")

if __name__ == "__main__":
    main()
