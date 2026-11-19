# -IntroProg_Lab2_P302_G5_U281274_U281566
Pràctica 2 Programació

# Written Report - Lab 2: Pong
**Autors:** Francisco Adama Riera (u281566) i Pau Pons Borrallo (u281274)

## 1. Team and contribution:
Ens hem distribuït la feina de manera igualitària:
- **Fran (u281566):** S'ha encarregat de les 6 primeres funcions (inclosa una d'extra), enfocades a la gestió d'entrades, validacions, preguntes a l'usuari i configuració inicial (`read_int_in_range`, `read_int_in_range_w_default`, `ask_yes_no`, `read_color`, `get_config` i `change_colors`).
- **Pau (u281274):** S'ha encarregat de les 4 últimes funcions de física, lògica de moviment, col·lisions i IA de la màquina (`move_paddle`, `collide_ball_paddle`, `reset_ball` i `compute_ai_direction`), així com de la seva integració al bucle principal (`main()`).

## 2. Overview of your design:
El programa funciona com un flux seqüencial on totes les dades viatgen mitjançant paràmetres i valors de retorn (tuples), sense utilitzar variables globals per modificar l'estat del joc:
1. **Fase de configuració:** A l'inici de `main()`, es crida `get_config()` (que al seu torn reutilitza `ask_yes_no()` i `read_int_in_range()`), retornant una tupla amb les mides i velocitats inicials. Tot seguit, `change_colors()` crida `read_color()` per obtenir les tuples RGB `(r, g, b)` del fons i dels elements.
2. **Bucle principal del joc (`while running:`):** Aquests valors inicials s'emmagatzemen en variables locals dins de `main()`. A cada fotograma, la posició de les pales s'actualitza passant la posició actual a `move_paddle()` (amb la direcció del teclat o de `compute_ai_direction()`), que retorna la nova coordenada `y`. De la mateixa manera, la posició i velocitat de la pilota es passen a `collide_ball_paddle()`, que retorna els nous valors actualitzats de `(x, y, vx, vy, hit)`. Quan es marca un punt, `reset_ball()` rep les velocitats base i retorna la pilota al centre.

## 3. Key implementation decisions

**Fran (Configuració i validació d'entrades):**
- **Validació al menú de configuració (`read_int_in_range`, `ask_yes_no`, `read_color`):** Per evitar que el programa falli si l'usuari introdueix lletres o números fora de rang, fem servir bucles `while True` combinats amb `try / except ValueError` i comprovacions `low <= val <= high`. A `ask_yes_no()` convertim l'entrada a minúscules amb `.lower()` per acceptar qualsevol variant de `y/yes/n/no`. A `read_color()` cridem tres vegades `read_int_in_range()` entre `0` i `255` i retornem la tupla `(r, g, b)` que necessita Pygame.
- **get_config() i change_colors():** Pregunten a l'usuari amb `ask_yes_no()` si vol personalitzar els paràmetres o els colors; en cas negatiu retornen directament els valors per defecte, i en cas afirmatiu van demanant cada dada validada.
- **Diferències respecte als docstrings:** Hem afegit la funció auxiliar extra `read_int_in_range_w_default(prompt, low, high, default_val)`, que comprova primer si l'usuari ha premut ENTER (`""`) abans de convertir a enter per retornar el valor per defecte sense obligar a escriure cada paràmetre.

**Pau (Col·lisions i moviment):**
- **Col·lisió pilota-pala i prevenció d'enganxament (`collide_ball_paddle`):** Per detectar el xoc, calculem les quatre vores de la pilota sumant i restant el seu radi (`x ± radius`, `y ± radius`) i comprovem el solapament horitzontal i vertical amb el rectangle de la pala. Per evitar que la pilota quedi atrapada rebotant dins la pala en fotogrames consecutius, a més d'invertir `vx`, recol·loquem `x` just a la vora exterior de la pala (`paddle_right + radius` o `paddle_left - radius`).
- **Moviment de les pales i serveis (`move_paddle`, `reset_ball`):** A `move_paddle()` multipliquem la direcció (`-1`, `0`, `1`) per la velocitat i acotem el resultat entre `0` i `HEIGHT - paddle_h`. A `reset_ball()`, decidim el sentit horitzontal segons `to_right` (`vx0` o `-vx0`) i apliquem una variació aleatòria a `vy` amb `rnd.uniform(0.7, 1.5) * rnd.choice((-1, 1))`.
- **Comportament de la màquina (`compute_ai_direction`):** Calculem el centre de la pala (`paddle_y + paddle_h / 2`) i definim una franja central de tolerància perquè la màquina no sigui invencible ni vibri constantment quan la pilota s'alinea.

## 4. Problems and solutions:
- **La pala se sortia per la part inferior de la pantalla:** Al principi comparàvem `current_y > HEIGHT`, però vam veure que `current_y` representa la vora superior de la pala i no la inferior. Ho vam solucionar fixant el límit inferior a `HEIGHT - paddle_h`.
- **Imprecisió en els rebots contra parets i pales:** Inicialment comprovàvem les col·lisions fent servir només el centre `(x, y)` de la pilota, cosa que feia que mitja pilota entrés dins la paret o la pala abans de rebotar. Ho vam corregir fent servir les quatre vores amb el radi (`x - radius`, `x + radius`, `y - radius`, `y + radius`) i ajustant la posició al límit en el moment del xoc.
- **La pilota no rebotava per un error d'operador (`==` vs `=`):** En detectar la col·lisió havíem escrit `hit == True` (comparació) en lloc de `hit = True` (assignació), de manera que la funció sempre retornava `False` i la pilota travessava les pales.
- **Errors amb entrades buides i no numèriques (Fran):** A `read_int_in_range_w_default()`, intentar convertir `""` a `int` feia saltar un `ValueError`; ho vam resoldre comprovant la cadena buida abans del bloc `try/except`.

## 5. Testing:
Hem verificat cada part de manera incremental:
- **Menú de configuració:** Hem introduït textos invàlids, números negatius, valors per sobre del màxim, majúscules/minúscules i tecles ENTER buides per comprovar que el programa demana de nou la dada sense tancar-se.
- **Joc i col·lisions:** Hem provat tant el mode 2 jugadors com el mode contra la màquina amb diferents mides de pala i velocitats, forçant rebots a les cantonades superiors i inferiors de les pales i comprovant que ni les pales ni la pilota surten dels marges ni perden velocitat.

## 6. Optional features:
- **Angle de rebot variable segons el punt d'impacte:** A `collide_ball_paddle()`, calculem la desviació normalitzada (`offset`) de la pilota respecte al centre de la pala i modifiquem lleugerament `vy` perquè els cops prop de les puntes obrin més l'angle de trajectòria.
- **Variació dinàmica de velocitat en els rebots:** Apliquem un multiplicador aleatori amb `rnd.choice()` sobre `vx` en rebotar a la pala, acotant la velocitat dins d'un rang mínim i màxim perquè la partida sigui més dinàmica sense arribar a ser injugable ni massa lenta.

## 7. Use of AI tools:
S'ha fet servir una eina d'IA (Gemini) per revisar casos límit de validació d'entrades, millorar la redacció dels comentaris i del report per aportar més claredat, i per buscar funcions dins del mòdul `random` (`rnd.choice`, `rnd.uniform`) per fer el joc més dinàmic. Tots els suggeriments s'han provat manualment abans de ser implementats definitivament. Finalment, havíem fet el report en un document Word i hem utilitzat Gemini per a que ens convertis el report en format de Github sense perdre el format que havíem creat a Word.
