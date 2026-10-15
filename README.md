# DRASOL — Gestion des livraisons

Application React/Vite + Supabase pour remplacer l'assignation des livraisons sur WhatsApp.

## Fonctionnalités
- 3 rôles : Admin, Closeuse, Livreur.
- Connexion par téléphone + mot de passe.
- Closeuse : commandes nouvelles/assignées, recherche, assignation à un livreur.
- Livreur : commandes assignées uniquement, Appeler, WhatsApp, En cours, Livré, Non livré + raison.
- Admin : dashboard du jour, toutes les commandes, filtres, gestion des closeuses/livreurs.
- Supabase Auth + PostgreSQL + RLS.
- Edge Function sécurisée pour créer/supprimer les comptes d'équipe avec la Service Role Key côté serveur uniquement.
- Interface responsive mobile.

## 1. Créer le projet Supabase
1. Crée un projet sur Supabase.
2. Dans **SQL Editor**, colle tout le contenu de `supabase/schema.sql` et exécute-le.
3. Va dans **Authentication > Users > Add user** et crée manuellement le premier compte Admin avec un email temporaire quelconque et un mot de passe.
4. Récupère son UUID puis décommente la dernière ligne `insert into public.profiles...` dans `schema.sql`, remplace `USER_UUID`, puis exécute-la. Exemple :
   `insert into public.profiles(id,name,phone,role) values ('UUID','Admin','90000000','admin');`
5. Le téléphone utilisé dans `profiles` doit être celui utilisé pour se connecter.

## 2. Déployer la fonction Admin
Installer Supabase CLI puis, dans ce dossier :

```bash
supabase login
supabase link --project-ref TON_PROJECT_REF
supabase secrets set SUPABASE_SERVICE_ROLE_KEY="TA_SERVICE_ROLE_KEY"
supabase functions deploy manage-user --no-verify-jwt
```

La fonction vérifie elle-même le JWT et le rôle Admin avant toute opération. **Ne mets jamais la Service Role Key dans le frontend.**

## 3. Installer et lancer le frontend

```bash
npm install
cp .env.example .env
```

Dans `.env` :

```env
VITE_SUPABASE_URL=https://TON-PROJET.supabase.co
VITE_SUPABASE_ANON_KEY=TA_CLE_ANON
```

Puis :

```bash
npm run dev
```

## 4. Déploiement Vercel / Netlify
Build command : `npm run build`
Output directory : `dist`

Ajouter les deux variables `VITE_SUPABASE_URL` et `VITE_SUPABASE_ANON_KEY` dans les variables d'environnement du service de déploiement.

## 5. Ajouter les commandes
Le site de gestion suppose que les commandes sont déjà insérées dans la table `orders` par ton système e-commerce/API. Exemple :

```sql
insert into public.orders(customer_name,customer_phone,address,order_items,amount)
values (
  'Koffi Mensah',
  '90123456',
  'Agoè, Lomé',
  '[{"name":"T-shirt noir","quantity":2},{"name":"Casquette","quantity":1}]',
  25000
);
```

L'intégration avec la boutique e-commerce peut ensuite appeler Supabase pour créer automatiquement chaque nouvelle commande.

## Important sur la connexion
Le frontend transforme le numéro en identifiant email interne du type `90000000@login.drasol.local`. L'utilisateur ne voit jamais cet email : il se connecte simplement avec son numéro + mot de passe.
