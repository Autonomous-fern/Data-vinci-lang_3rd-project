# ___________________OBJECTIF : Constituer une base de données des agences de mannequinat homme à Paris et Londres___________________________________________
"""0 fonctions, 0 classes, c'est volontaire"""

import gspread  # module pour la gestion de l'api google
import requests  # module pour récupérer le code html d'une page
from bs4 import BeautifulSoup  # module pour lire l'html d'une page
import time  # nous permet de temporiser les requêtes à l'api dans une loop


# _______Initialisation du scraping_____________________________________________
URL_UK1 = "https://mannequinat.fr/liste/agences-de-mannequins/?pays=royaume-uni&spe_mannequin=hommes"
URL_UK2 = "https://mannequinat.fr/liste/agences-de-mannequins/page/2/?pays=royaume-uni&spe_mannequin=hommes#038;spe_mannequin=hommes"
URL_FR1 = "https://mannequinat.fr/liste/agences-de-mannequins/?pays=france&spe_mannequin=hommes"
URL_FR2 = "https://mannequinat.fr/liste/agences-de-mannequins/page/2/?pays=france&spe_mannequin=hommes#038;spe_mannequin=hommes"
URL_FR3 = "https://mannequinat.fr/liste/agences-de-mannequins/page/3/?pays=france&spe_mannequin=hommes#038;spe_mannequin=hommes"
Input_URLs_list = [URL_UK1, URL_UK2, URL_FR1, URL_FR2, URL_FR3]

Output_URLs_list = []  # on va stocker les URL des pages auxquelles on veut accéder pour les scraper par la suite en itérant sur les différentes pages associées chacunes à un unique élément de la data base à rendre

for url in Input_URLs_list:
    page = requests.get(url)
    soup = BeautifulSoup(page.content, "html.parser")
    ugly_listing = soup.find(id="pros_listing")  # on récupère les éléments sous la balise d'ID "pros_listing"
    a_tags_in_ugly_listing = ugly_listing.find_all("a")  # parmis ceux trouvées on accède à ceux qui sont sous la balise a

    for a_tags in a_tags_in_ugly_listing:
        Output_URL_i = a_tags.get("href")  # parmis a_tags_in_ugly_listing on récupère les valeurs href pour accéder aux url des pages uniques qui comportent les données qu'ont veux pour chaque élément de la data base à constituer
        Output_URLs_list.append(Output_URL_i)

# ________Scraping______________________________________________________________

data_base = []
for url in Output_URLs_list:
    try:
        page = requests.get(url)  # on parcourt les urls uniques qui contiennet les données à scrapper par éléments
        soup = BeautifulSoup(page.content, "html.parser")
        agency_name = soup.find("h1", class_="ppro_name ff_serif_ital").string  # les noms de classes sont liés à des éléments uniques sur ce site donc on peut faire une recherche par l'id "ppro_name..."
        infos_tag = soup.find("div", class_="ppro_infos").find_all("span", class_="info")  # ici on n'a pas de nom de classe unique qui nous permet d'accéder directement à l'élément donc on passe par une catégorie puis sa descendance
        ville_pays = infos_tag[0].next_sibling  # next_sibling nous permet de récupérer le text APRES la fermeture du tag
        adresse = infos_tag[1].next_sibling
        telephone = infos_tag[2].next_sibling
        genres = infos_tag[3].next_sibling
        website = infos_tag[4].next_sibling.next_sibling.get_text()  # le site internet est inscrit différement des autres infos car le lien est cliquable donc la méthode pour le récupérer n'est pas la même

        contact_i = [agency_name, ville_pays, adresse, telephone, genres, website]  # on a notre élément au complet
        data_base.append(contact_i)  # on ajoute l'élément à la data base afin de tout copier sur le gsheet
    except Exception:
        pass  # try...except nous permet d'ignorer les url potentiellement scrappés qui ne correspondent pas au format d'élément que l'on recherche et qui n'est donc pas un élément de la database.


# ________Connection à notre gsheet_____________________________________________
sa = gspread.service_account(filename="vici-language-project-1dc3049324f2.json")  # ici on se connecte à l'api google via une clée téléchargée et stocjée dans le json
sh = sa.open("Male model agencies Paris/London")  # ouverture du fichier gsheet
ws = sh.worksheet("DATA")  # ouverture du worksheet 1

# ________Ecriture dans le gsheet_______________________________________________
# (OPTI POTENTIELLE 1 : on veut rajouter les NOUVEAUX éléments uniquement afin de minimiser le nombre de requête en cas de modif de la liste)
# (OPTI POTENTIELLE 2 : on ajoute les éléments par ligne avec 1 seule boucle for.?? ou pas du tout de boucle et on copie colle la range directement dans le sheet > peut nous faire gagner pas mal de temps)

last_row = len(data_base)
last_column = len(data_base[0])
for i in range(last_row):
    for j in range(last_column):
        ws.update_cell(i + 2, j + 1, data_base[i][j])
        time.sleep(1.01)  # on est limité à 60 requête/min/utilisateur pour l'API google

# ===> runtime = 793.233s (très bof) mais la base de donnée est ok (dispo ici en accès à la demande: https://docs.google.com/spreadsheets/d/1_cxCLggYuWZwb3z1Dg1YC9DXfeS-3WKf15lRxJP_6I4/edit#gid=0)
