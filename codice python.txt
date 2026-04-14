import psycopg2
from datetime import date

def connetti_db():
    try:
        return psycopg2.connect(
            host="localhost",
            database="postgres",
            user="postgres",
            password="admin" 
        )
    except Exception as e:
        print(f"Errore: {e}")
        return None

    
def mostra_benvenuto():
    print("SISTEMA SPESE PERSONALI")
    print("\n" + ""*20)
    print("Benvenuto!")
    print(""*5)
    print("Questo software ti permette di monitorare le tue spese e")
    print("il tuo budget.")

        
def aggiungi_categoria():
    while True:
        nome = input("\nInserisci il nome di una nuova categoria: ").strip()
        if not nome:
            print("❌ Errore: Il nome della categoria non può essere vuoto!")
            continue
        
        conn = connetti_db()
        if conn:
            cur = conn.cursor()
            try:
                cur.execute("INSERT INTO Categorie (nome) VALUES (%s)", (nome,))
                conn.commit()
                print(f"✅ Categoria '{nome}' inserita correttamente!")
                break
            except Exception as e:
                print(f"❌ Errore: la categoria esiste già. Riprova")
            finally:
                conn.close()
        else:
            break

            
def inserisci_spesa():
    conn = connetti_db()
    if not conn: return
    cur = conn.cursor()

    try:
        while True:
            cur.execute("SELECT * FROM Categorie")
            cats = cur.fetchall()
            print("\nCategorie disponibili:")
            for c in cats: print(f"{c[0]}: {c[1]}")
        
            try:
                nome_cat = input("Scegli il nome della categoria: ")
                cur.execute("SELECT id_categoria FROM Categorie WHERE nome = %s", (nome_cat,))
                risultato = cur.fetchone()

                if risultato:
                    id_cat = risultato[0]
                    break
                else:
                    print(f"❌ Errore: La categoria '{nome_cat}' non esiste. Riprova.")

            except ValueError:
                print("❌ Inserisci un numero per l'ID!")
            
        data_input = input("Inserisci la data (YYYY-MM-DD) [Premi Invio per oggi]: ")
        if data_input == "":
            data_spesa = date.today()
        else:
            data_spesa = data_input
                
        while True:
            try:
                importo_spesa = float(input("Inserisci l'importo della spesa: ").replace(',', '.'))
                if importo_spesa  <=0:
                    print("❌ Errore: L'importo deve essere maggiore di zero!")
                else:
                    break
            except ValueError:
                print("❌ Errore: Inserisci un numero valido.")
           
        desc = input("Descrizione: ")

        cur.execute("INSERT INTO Spese (data, importo, id_categoria, descrizione) VALUES (%s, %s, %s, %s)",
                    (data_spesa, importo_spesa, id_cat, desc))
    
        conn.commit()
        print("✅ Spesa inserita correttamente!")

    except Exception as e:
        print(f"❌ Errore: {e}")
    finally:
        conn.close()


def definisci_budget():
    conn = connetti_db()
    if not conn: return
    cur = conn.cursor()
    
    try:
        
        print("\nIMPOSTAZIONE BUDGET")
        mese_anno = input("Inserisci il periodo (YYYY-MM): ")
        while True:
            cur.execute("SELECT * FROM Categorie")
            cats = cur.fetchall()
            print("\nCategorie disponibili:")
            for c in cats: print(f"{c[1]}")
            nome_cat = input("Scegli la categoria: ")
            cur.execute("SELECT id_categoria FROM Categorie WHERE nome = %s", (nome_cat,))
            risultato = cur.fetchone()
            if risultato:
                id_cat = risultato[0]
                break
            else:
                print(f"❌ Errore: La categoria '{nome_cat}' non esiste. Riprova.")
        while True:
            try:
                importo_raw = input("Inserisci l'importo del budget: ").replace(',', '.')
                importo_budget= float(importo_raw)
                if importo_budget <= 0:
                    print("❌ Errore: L'importo del budget non è ammesso!")
                else:
                    break
            except ValueError:
                print("❌ Errore: Inserisci un numero valido.")
                

        cur.execute("""
            INSERT INTO Budget (id_categoria, importo_limite, anno_mese)
            VALUES (%s, %s, %s)
            ON CONFLICT (id_categoria, anno_mese) 
            DO UPDATE SET importo_limite = EXCLUDED.importo_limite
        """, (id_cat, importo_budget, mese_anno))
        
        conn.commit()
        
        print("✅ Budget mensile salvato correttamente.")

    
    except Exception as e:
        print(f"❌ Errore durante l'elaborazione: {e}")
    finally:
        conn.close()

def report_totale_per_categoria():
    conn = connetti_db()
    cur = conn.cursor()
    query = """
        SELECT c.nome, SUM(s.importo) 
        FROM Spese s
        JOIN Categorie c ON s.id_categoria = c.id_categoria
        GROUP BY c.nome
    """
    cur.execute(query)
    risultati = cur.fetchall()
    print("\n--- TOTALE SPESE PER CATEGORIA ---")
    print("="*30)
    for r in risultati:
        print(f"{r[0]}: {r[1]:.2f}€")
    conn.close()

def report_spese_vs_budget():
    conn = connetti_db()
    cur = conn.cursor()
    periodo = input("Inserisci anno e mese (YYYY-MM): ").strip()
    query = """
        SELECT c.nome,
            b.importo_limite,
            COALESCE(SUM(s.importo), 0)
        FROM Budget b
        JOIN Categorie c ON b.id_categoria = c.id_categoria
        LEFT JOIN Spese s ON c.id_categoria = s.id_categoria 
             AND TO_CHAR(s.data, 'YYYY-MM') = TRIM(b.anno_mese)
        WHERE b.anno_mese = %s
        GROUP BY c.nome, b.importo_limite
    """
    try:
        cur.execute(query, (periodo,))
        risultati = cur.fetchall()
        print(f"\n BUDGET VS SPESE ")
        if not risultati:
            print(f"Nessun budget trovato per il periodo {periodo}.")
        else:
            for r in risultati:
                nome, budget, speso = r
                differenza = budget - speso
                status = "✅" if differenza >= 0 else "⚠SUPERAMENTO BUDGET"
                print(f"{nome:<15} | Budget: {budget:>8.2f} | Speso: {speso:>8.2f} | Residuo: {differenza:>8.2f} {status}")
    except Exception as e:
        print(f"❌ Errore: {e}")
    finally:
        conn.close()

def report_elenco_spese():
    conn = connetti_db()
    cur = conn.cursor()
    cur.execute("""
        SELECT s.data, s.importo, c.nome, s.descrizione 
        FROM Spese s
        JOIN Categorie c ON s.id_categoria = c.id_categoria
        ORDER BY s.data DESC
    """)
    risultati = cur.fetchall()
    print("\n--- ELENCO COMPLETO SPESE ---")
    print(f"{'Data':<10} | {'Categoria':<12} | {'Importo':>9} | {'Descrizione'}")
    print("-" * 60)
    for r in risultati:     
        print(f"{str(r[0]):<10} | {r[2]:<12} | {r[1]:>8.2f}€ | {r[3]}")
    conn.close()
    
def menu():
    while True:
        print("\n" + "="*30)
        print("1. Gestione Categorie ")
        print("2. Inserisci Spesa ")
        print("3. Definisci Budget Mensile")
        print("4. Visualizza Report")
        print("5. Esci")
        
        scelta = input("\nInserisci la tua scelta: ")
        
        if scelta == '1': aggiungi_categoria()
        elif scelta == '2': inserisci_spesa()
        elif scelta == '3': definisci_budget()
        elif scelta == '4': sottomenu_report()
        elif scelta == '5':
            print("Chiusura in corso... Arrivederci!")
            break
        else:
            print(""*5)
            print("Scelta non valida. Riprovare")

def sottomenu_report():
    while True:
        print("\n" + ""*30)
        print("MENU DEI REPORT")
        print("\n" + ""*20)
        print("1. Totale spese per categoria")
        print("2. Spese mensili vs budget")
        print("3. Elenco completo delle spese ordinate per data")
        print("4. Ritorna al menu principale")
        
        scelta = input("\nInserisci la tua scelta: ")
        
        if scelta == '1':
            report_totale_per_categoria()
        elif scelta == '2':
            report_spese_vs_budget()
        elif scelta == '3':
            report_elenco_spese()
        elif scelta == '4':
            break
        else:
            print(""*5)
            print("Scelta non valida. Riprovare")

if __name__ == "__main__":
        mostra_benvenuto()
        menu()
