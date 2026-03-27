# Write-Up : Deep Filter (HackDay ESIEE)

**Plateforme :** HackDay  
**Catégorie :** Stéganographie  
**Difficulté :** Moyen  

## Objectif
L'objectif est d'extraire un **flag caché** dans une image PNG nommée `logo.png`.

## Analyse de l'énoncé
L'énoncé fournit plusieurs indices : 

* **Titre :** "FILTRE PROFOND" suggère une manipulation au cœur de la structure du fichier.

* **Indice textuel :** La mention d'un "messager de données de bas niveau" pointe vers une manipulation au niveau des bits ou des structures internes.

* **Intégrité :** Le hash SHA256 du fichier est 3339ce838ca957942caa346850aeb8c89554991b193bbe02521f911c4116a9b.

## Exploration initiale

Plusieurs outils classiques ont été utilisés sans succès pour trouver le flag en clair :

* **Binwalk :** binwalk -e logo.png a extrait des données en zlib qui n'ont pas révélé le flag.

* **Strings :** Aucune chaîne de caractères lisible n'a été trouvée via strings.

* **File :** La commande file identifie simplement le fichier comme de la donnée brute après extraction.

## Analyse technique des filtres

L'analyse s'est portée sur les ** scanline filters ** du format PNG. Chaque ligne d'une image PNG utilise l'un des 5 filtres de compression (0 à 4).

En utilisant pngcheck -vv logo.png, on observe que seuls les filtres ** 1 ** et ** 2 ** sont utilisés pour les 275 lignes de l'image:

* **Filtre 1** = Bit **0** 

* **Filtre 2** = Bit **1**

## Automatisation (Python)

Pour décoder le message caché dans ces filtres, j'ai utilisé le script suivant :

```python
import zlib
import struct

def extract_flag(filename):
    with open(filename, 'rb') as f:
        # Sauter la signature PNG
        f.read(8)
        idat_data = b""
        while True:
            chunk_len_bin = f.read(4)
            if not chunk_len_bin:
                break
            chunk_len = struct.unpack('>I', chunk_len_bin)[0]
            chunk_type = f.read(4)
            if chunk_type == b'IDAT':
                idat_data += f.read(chunk_len)
            elif chunk_type == b'IEND':
                break
            else:
                f.read(chunk_len)
            f.read(4) # Sauter le CRC
        
        # Décompression des données
        data = zlib.decompress(idat_data)

        # Dimensions de l'image (250x275, RGB = 3 bytes par pixel)
        width, height = 250, 275
        row_size = (width * 3) + 1

        # Extraction du premier octet de chaque ligne (le filtre)
        filters = [data[i] for i in range(0, len(data), row_size)]

        # Conversion 1 -> 0 et 2 -> 1
        binary_str = "".join(['0' if f == 1 else '1' for f in filters if f in [1, 2]])

        # Conversion binaire vers texte
        flag = "".join(chr(int(binary_str[i:i+8], 2)) for i in range(0, len(binary_str), 8))
        return flag

print("Le flag est :", extract_flag('logo.png'))
```

## Résultat

Le script nous donne le flag final :

**HACKDAY{S0_MuCh_D4T4_!N_PnG}** 

## Conclusion

Ce challenge illustre une technique de stéganographie avancée où l'information est dissimulée dans les **métadonnées de structure** du fichier (les filtres de compression) plutôt que dans les pixels eux-mêmes. Cela permet de cacher des données sans altérer l'aspect visuel de l'image.
