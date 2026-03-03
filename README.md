# Agentic-RAG
### Megoldás
A notebookban egy Agentic RAG alapú megoldást implementáltam, amely egy adott kérdésre ad választ a kapott PDF alapján. Amennyiben a kérdés nem releváns a témához (mely döntést az LLM hozza meg), úgy nem ad az ágens érdemi választ. Ha azonban releváns, akkor a vektoradatbázisból a szöveg lényegi részeit kinyerjük a kérdés embeddingje alapján. A kérdést és a kontextust felhasználva állítja elő az LLM a válaszát. A válasz minőségét ellenőrzi ezután. Amennyiben az megfelelő, úgy az lesz a végső válasz, ellenkező esetben javítja azt. Ha egy maximum értéknél többször kell javítani a választ, akkor az ágens a váalszában jelzi, hogy nem tudta létrehozni megfelelő minőségben a választ.

A megoldás lokális GPU-t feltételez, az LLM-et az Ollama könytárral futtatja. A reprodukálhatóság érdekében az LLM temperature értéket 0-ra állítottam. A kód Google Colab notebook-ban lett tesztelve. 

### Notebook felépítése:

1.   **Setup:** könyvtárak, dependenciák letöltése, Ollama indítása a lokális LLM-futtatáshoz.
2.   **Importálás:** könyvtárak importálása, paraméterek beállítása, modell betöltése.
3.   **Adatfeldolgozás:** PDF letöltése, chunkolása, vektoradatbázis elkészítése. A chunkokat egy nyílt forráskódú (open-source) embedding modellel alakítja vektorokká, majd a FAISS könyvtárral hoztam létre a vektoradatbázist.
4. **Agent és a gráf létrehozása:** definiáltam az egyes node-okat és az útvonalválasztáshoz szükséges kondíciós függvényeket, majd felépítettem a gráfot. Ezt vizuálisan is megjelenítettem.
5. **Tesztelés:** néhány egyszerű teszttel validáltam az ágens helyes működését. Egy releváns és egy nem releváns kérdést tettem fel, melyekre az ágens helyesen, az elvártnak megfelelően válaszolt.

### Tervezési döntések, alkalmazott architektúra:

*   A Qwen2.5-14B nagy nyelvi modellt alkalmaztam, amely használható egy 16GB-os GPU-n. Lokálisan futtattam, a reprodukálhatóság érdekében a temperature értéket 0-ra állítottam.
*   Vektoradatbázis és Embedding (FAISS + all-MiniLM-L6-v2): Ezek a nyílt forráskódú, erőforrás-hatékony megoldások, megfelelnek a minőségi, illetve skálázhatósági elvárásoknak.
*   LangGraph framework: Az elvárásokban rögzített keretrendszer.

### Bottleneck-ek:

*   Egy kérdés megválaszolása sok LLM futtatást is igényelhet, attól függően, hányszor kell újragenerálni a választ.
*   A jelenlegi, karakteralapú darabolást alkalmazó chunking stratégia elég statikus, megszakíthat összefüggő részeket.
* A válasz minőségének ellenőrzésének lépése (quality check) egyelőre nem elég robosztus a hallucinációval szemben.

### Tesztelés:

*   RAGAs: Hallucináció, válasz relevanciájának és kinyerés pontosságának mérése.
*   Node-ok tesztelése unit tesztekkel.
*   Egyes lépések, válaszok generálásának idejének mérése.

### Továbbfejlesztési lehetőségek:

*   Retrieval fejlesztése: kulcsszó alapú keresés vagy Reranking.
*   Karakteralapú chunking helyett a jelentés mentén (pl bekezdés alapján) szétválasztás.
* Structured Outputs használata a válasz string-jében megjelenő "igen" vagy "nem" szavak keresése helyett.
* Feladat komplexitása alapján szét lehetne osztani a feladatokat: a könnyebb task-okat egy kisebb modell végezné.
* A válasz újragenerálásakor (regenerate_answer) új kontextust is meg lehetne adni, hiszen lehet a rossz korábbi kontextus okozta a rossz választ. Ezt az új kontextust egy (LLM-mel) újrafogalmazott query-vel lehetne kinyerni. 
