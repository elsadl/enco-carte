# Mise à jour des données

Chaque script génère deux fichiers :
- **scriptEntreprises** → `dict_entreprises.json` (le lien entre le nom des entreprises et leur id) et `entreprises.json` (les données utilisées pour générer les pages entreprises)
- **scriptPays** → `dict_pays.json` (les traductions des noms de pays, pour l'internationalisation) et `nuts_with_data.json` (le fond de carte de l'Europe avec les données de santé propres à chaque pays)

### Pour générer les fichiers de données
1. Les scripts et les données doivent être organisés comme dans ce repo
2. Dans un terminal, on vérifie que python et pip (package manager) sont installés
````
python --version
````
````
python -m pip --version
```` 
Si ce n'est pas le cas, voir [la doc de pip](https://pip.pypa.io/en/stable/getting-started/)

3. Si ce n'est pas fait, on installe [openpyxl](https://pypi.org/project/openpyxl/) (la librairie qui permet la lecture des tableaux excel)
````
pip install openpyxl
````
4. On se place dans le dossier où se trouvent les scripts
````
cd path/to/folder
````
Puis on lance les scripts
- Si on veut mettre à jour les données de la carte
````
python scriptPays.py
````
- Si on veut mettre à jour les données des entreprises
````
python scriptEntreprises.py
````
5. Les fichiers mis à jour sont générés dans le dossier `data`, il n'y a plus qu'à les remplacer dans le dossier `public/data` du site
