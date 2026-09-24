# Directives de sécurité

## Vérifications de sécurité obligatoires

Avant TOUT commit :
- [ ] Pas de secrets en dur (clés API, mots de passe, tokens)
- [ ] Toutes les entrées utilisateur validées
- [ ] Prévention des injections SQL (requêtes paramétrées)
- [ ] Prévention XSS (HTML assaini)
- [ ] Protection CSRF activée
- [ ] Authentification/autorisation vérifiée
- [ ] Limitation de débit sur tous les endpoints
- [ ] Les messages d'erreur ne fuient pas de données sensibles

## Gestion des secrets

- NE JAMAIS coder les secrets en dur dans le code source
- TOUJOURS utiliser des variables d'environnement ou un gestionnaire de secrets
- Valider que les secrets requis sont présents au démarrage
- Faire tourner (rotate) tout secret ayant pu être exposé

## Protocole de réponse en cas d'incident de sécurité

Si un problème de sécurité est trouvé :
1. ARRÊTER immédiatement
2. Utiliser l'agent **security-reviewer**
3. Corriger les problèmes CRITIQUES avant de continuer
4. Faire tourner (rotate) tout secret exposé
5. Revoir l'ensemble du code pour des problèmes similaires
