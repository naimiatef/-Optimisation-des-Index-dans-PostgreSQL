# -Optimisation-des-Index-dans-PostgreSQL

**Résumé en français :**

Cet article explique les différents types d'index dans PostgreSQL, leurs usages, les erreurs courantes à éviter, ainsi que des cas particuliers comme les index partiels, les expressions indexées, ou encore les index multi-colonnes.

---

### **Types d'index :**

1. **B-tree** :  
   L'index par défaut, adapté pour des recherches de valeurs spécifiques, des plages, ou des tris.  
   Syntaxe :  
   `CREATE INDEX idx_tree1 ON table(column_name);`

2. **Hash** :  
   Optimal pour les comparaisons d'égalité.  
   Syntaxe :  
   `CREATE INDEX idx_hash1 ON table USING HASH(column_name);`

3. **GiST** (Generalized Search Tree) :  
   Conçu pour des données complexes comme les données géométriques.  
   Syntaxe :  
   `CREATE INDEX ON table USING gist(column_name);`

4. **SP-GiST** (Space-Partitioned GiST) :  
   Idéal pour les données déséquilibrées ou partitionnées.  
   Syntaxe :  
   `CREATE INDEX ON table USING spgist(column_name);`

5. **GIN** (Generalized Inverted Index) :  
   Adapté aux types composites (JSONB, tableaux, recherche textuelle).  
   Syntaxe :  
   ```sql
   CREATE EXTENSION pg_trgm;
   CREATE INDEX idx_gin ON table USING gin(column_name gin_trgm_ops);
   ```

6. **BRIN** (Block Range Index) :  
   Stocke des résumés de blocs (valeurs min et max), adapté aux grandes tables triées naturellement.  
   Syntaxe :  
   `CREATE INDEX idx_brin1 ON table USING BRIN(column_name);`

---

### **Erreurs courantes :**

1. **Créer des index sur toutes les colonnes** : Cela alourdit les écritures et consomme de l'espace disque inutilement.
2. **Ignorer la sélectivité des colonnes** : Les colonnes avec peu de valeurs uniques (faible sélectivité) ne bénéficient pas d'un index performant.
3. **Choisir un mauvais type d'index** : Chaque type d'index est optimisé pour des cas spécifiques (par exemple, GIN pour JSONB, BRIN pour grandes tables triées).
4. **Négliger le coût de maintenance des index** : Les index augmentent les temps d'insertion et nécessitent des opérations comme `VACUUM` et `REINDEX`.
5. **Ne pas mettre à jour les statistiques** : Lancer `ANALYZE` régulièrement permet d'obtenir des plans de requêtes optimaux.

---

### **Cas particuliers :**

1. **Index sur expressions** :  
   Exemples d'index sur le résultat d'une fonction :  
   ```sql
   CREATE INDEX test1_lower_col1_idx ON cust(lower(firstname));
   ```

2. **Index partiels** :  
   Index sur un sous-ensemble de la table :  
   ```sql
   CREATE INDEX orders_unbilled_index ON orders (amount) WHERE billed IS NOT TRUE;
   ```

3. **Index multi-colonnes** :  
   Combine plusieurs colonnes dans un index, idéal pour des requêtes combinées.  
   Syntaxe :  
   `CREATE INDEX idx_staff1 ON staff(firstname, salary);`

4. **Considérations pour les valeurs NULL** :  
   Les NULL sont placés en début d’index B-tree. On peut les exclure via un index partiel ou les reclasser avec des options comme :  
   ```sql
   CREATE INDEX test2_info_nulls_low ON test2 (info NULLS FIRST);
   ```

5. **Index couvrants (covering index)** :  
   Incluent des colonnes supplémentaires nécessaires pour éviter de lire les données dans le tableau :  
   ```sql
   CREATE INDEX idx_test1 ON pgbench_tellers(bid) INCLUDE(tid, tbalance);
   ```

---

### **Considérations générales :**

- **Combiner colonnes fréquemment utilisées** dans un seul index multi-colonnes.  
- **Limiter le nombre de colonnes dans les index** multi-colonnes pour éviter une surcharge de taille.  
- **Évaluer régulièrement les performances** des index et les reconstruire si nécessaire.  
- **Choisir le bon type d'index** en fonction des besoins spécifiques des requêtes et du type de données.
