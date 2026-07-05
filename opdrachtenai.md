Opdracht: azctl v0.1 — Graph token + users list
Doel: een Go CLI die authenticeert tegen Microsoft Graph via client credentials flow en gebruikers uit je dev-tenant toont.
Vereisten
1. Setup

Entra dev-tenant met app registration: application permission User.Read.All, admin consent verleend, client secret aangemaakt
Repo azctl in je GitLab, go mod init, README met één alinea: wat het is, hoe je het draait

2. Functioneel

azctl users list print alle gebruikers uit de tenant als tabel: DisplayName, UserPrincipalName, AccountEnabled
azctl users list --output json print dezelfde data als JSON-array
Configuratie uitsluitend via env vars: AZCTL_TENANT_ID, AZCTL_CLIENT_ID, AZCTL_CLIENT_SECRET. Ontbreekt er één → duidelijke foutmelding en exit code 1, geen panic

3. Technisch — verplicht

Token ophalen met net/http handmatig (POST naar https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token, scope https://graph.microsoft.com/.default). Geen azidentity, geen SDK.
Response unmarshalen naar een TokenResponse struct met json-tags
Graph-call: GET https://graph.microsoft.com/v1.0/users met Authorization: Bearer <token> header, response naar []User structs
Paging: als @odata.nextLink aanwezig is, volg je die tot alle users binnen zijn (maak minimaal 15 testusers aan in je tenant zodat je paging kúnt testen met $top=5)
Elke error afgehandeld en omhoog gepropageerd met context (fmt.Errorf("fetching token: %w", err)) — geen enkele _ op een error
Packagestructuur: cmd/ (main + cobra), internal/graph/ (client, types)

4. Testen — verplicht

Minimaal één test op de token-parse (json.Unmarshal van een voorbeeldresponse)
Minimaal één test op de paging-logica met httptest.NewServer die twee pagina's mockt
go test ./... groen, go vet ./... schoon