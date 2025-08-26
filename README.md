<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Překladač genetického kódu</title>
    <!-- Tailwind CSS pro snadné a rychlé stylování -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f1f5f9; /* Slate-100 */
        }
        .chain-element {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 4px;
        }
        .chain-element p {
            margin: 0;
            white-space: nowrap;
        }
    </style>
</head>
<body class="flex flex-col items-center justify-center min-h-screen p-4">

    <!-- Hlavní kontejner aplikace -->
    <div class="bg-white rounded-2xl shadow-xl p-8 max-w-lg w-full">

        <h1 class="text-3xl font-bold text-center text-blue-800 mb-8">Překladač genetického kódu 🧬</h1>

        <!-- Kontejner pro řetězec aminokyselin -->
        <div id="aminoAcidChain" class="flex flex-wrap items-center justify-center gap-2 mb-8 p-4 bg-blue-50 border border-blue-200 rounded-lg overflow-x-auto">
            <!-- Pevný iniciační kodon AUG -->
            <div class="chain-element px-3 py-1 bg-blue-200 text-blue-800 rounded-full font-semibold">
                <p>AUG (Met)</p>
            </div>
        </div>

        <!-- Vstupní sekce s políčkem a tlačítky -->
        <div class="flex flex-col sm:flex-row items-center justify-center gap-4 mb-8">
            <input id="codonInput" type="text" maxlength="3" placeholder="Zadejte kodon (např. UUC)"
                   class="w-full sm:w-1/2 p-3 text-center border-2 border-gray-300 rounded-lg focus:outline-none focus:border-blue-500 transition-colors duration-200 uppercase text-lg">
            <button id="translateBtn"
                    class="w-full sm:w-auto px-6 py-3 bg-green-500 text-white font-semibold rounded-lg shadow-md hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-400 focus:ring-opacity-75 transition-colors duration-200">
                Přeložit
            </button>
            <button id="resetBtn"
                    class="w-full sm:w-auto px-6 py-3 bg-red-500 text-white font-semibold rounded-lg shadow-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-400 focus:ring-opacity-75 transition-colors duration-200 hidden">
                Reset
            </button>
        </div>

        <!-- Gemini API tlačítka a výstupy -->
        <div class="flex flex-col gap-4 mb-8">
            <div class="flex flex-col sm:flex-row justify-center gap-4">
                <button id="explainBtn"
                        class="w-full sm:w-auto px-6 py-3 bg-purple-500 text-white font-semibold rounded-lg shadow-md hover:bg-purple-600 focus:outline-none focus:ring-2 focus:ring-purple-400 focus:ring-opacity-75 transition-colors duration-200" disabled>
                    Vysvětlit aminokyselinu ✨
                </button>
                <button id="continueBtn"
                        class="w-full sm:w-auto px-6 py-3 bg-indigo-500 text-white font-semibold rounded-lg shadow-md hover:bg-indigo-600 focus:outline-none focus:ring-2 focus:ring-indigo-400 focus:ring-opacity-75 transition-colors duration-200" disabled>
                    Pokračovat v řetězci ✨
                </button>
            </div>
            <div id="explanationContainer" class="bg-blue-50 border border-blue-200 rounded-lg p-4 mt-4 hidden">
                <p id="explanationText" class="text-sm text-gray-700"></p>
            </div>
        </div>
        
        <!-- Sekce pro zobrazení výsledku a chyb -->
        <div class="bg-gray-100 border-2 border-gray-300 rounded-lg p-6 text-center">
            <p id="resultText" class="text-xl font-bold text-gray-700">Zadejte kodon a začněte tvořit!</p>
        </div>
    </div>

    <script>
        // JavaScript pro logiku aplikace
        document.addEventListener('DOMContentLoaded', () => {

            // Objekt, který mapuje kodony na aminokyseliny
            const geneticCode = {
                "UUU": "Fenylalanin (Phe)", "UUC": "Fenylalanin (Phe)",
                "UUA": "Leucin (Leu)", "UUG": "Leucin (Leu)", "CUU": "Leucin (Leu)", "CUC": "Leucin (Leu)", "CUA": "Leucin (Leu)", "CUG": "Leucin (Leu)",
                "AUU": "Isoleucin (Ile)", "AUC": "Isoleucin (Ile)", "AUA": "Isoleucin (Ile)",
                "AUG": "Methionin (Met)",
                "GUU": "Valin (Val)", "GUC": "Valin (Val)", "GUA": "Valin (Val)", "GUG": "Valin (Val)",
                "UCU": "Serin (Ser)", "UCC": "Serin (Ser)", "UCA": "Serin (Ser)", "UCG": "Serin (Ser)", "AGU": "Serin (Ser)", "AGC": "Serin (Ser)",
                "CCU": "Prolin (Pro)", "CCC": "Prolin (Pro)", "CCA": "Prolin (Pro)", "CCG": "Prolin (Pro)",
                "ACU": "Threonin (Thr)", "ACC": "Threonin (Thr)", "ACA": "Threonin (Thr)", "ACG": "Threonin (Thr)",
                "GCU": "Alanin (Ala)", "GCC": "Alanin (Ala)", "GCA": "Alanin (Ala)", "GCG": "Alanin (Ala)",
                "UAU": "Tyrosin (Tyr)", "UAC": "Tyrosin (Tyr)",
                "UAA": "STOP", "UAG": "STOP", "UGA": "STOP",
                "CAU": "Histidin (His)", "CAC": "Histidin (His)",
                "CAA": "Glutamin (Gln)", "CAG": "Glutamin (Gln)",
                "AAU": "Asparagin (Asn)", "AAC": "Asparagin (Asn)",
                "AAA": "Lysin (Lys)", "AAG": "Lysin (Lys)",
                "GAU": "Kyselina asparagová (Asp)", "GAC": "Kyselina asparagová (Asp)",
                "GAA": "Kyselina glutamová (Glu)", "GAG": "Kyselina glutamová (Glu)",
                "UGU": "Cystein (Cys)", "UGC": "Cystein (Cys)",
                "UGG": "Tryptofan (Trp)",
                "CGU": "Arginin (Arg)", "CGC": "Arginin (Arg)", "CGA": "Arginin (Arg)", "CGG": "Arginin (Arg)", "AGA": "Arginin (Arg)", "AGG": "Arginin (Arg)",
                "GGU": "Glycin (Gly)", "GGC": "Glycin (Gly)", "GGA": "Glycin (Gly)", "GGG": "Glycin (Gly)",
            };

            // Získání odkazů na HTML prvky
            const codonInput = document.getElementById('codonInput');
            const translateBtn = document.getElementById('translateBtn');
            const resultText = document.getElementById('resultText');
            const aminoAcidChain = document.getElementById('aminoAcidChain');
            const resetBtn = document.getElementById('resetBtn');
            const explainBtn = document.getElementById('explainBtn');
            const continueBtn = document.getElementById('continueBtn');
            const explanationContainer = document.getElementById('explanationContainer');
            const explanationText = document.getElementById('explanationText');

            // Funkce pro přidání aminokyseliny do řetězce
            function addAminoAcidToChain(aminoAcid, isStop) {
                // Vytvoření nového elementu pro šipku
                const arrowEl = document.createElement('span');
                arrowEl.textContent = '→';
                arrowEl.classList.add('text-gray-400');
                aminoAcidChain.appendChild(arrowEl);

                // Vytvoření nového elementu pro aminokyselinu
                const aminoAcidEl = document.createElement('div');
                aminoAcidEl.classList.add('chain-element', 'px-3', 'py-1', 'rounded-full', 'font-semibold');
                
                if (isStop) {
                    aminoAcidEl.classList.add('bg-red-200', 'text-red-800');
                } else {
                    aminoAcidEl.classList.add('bg-green-200', 'text-green-800');
                }
                aminoAcidEl.innerHTML = `<p>${aminoAcid}</p>`;
                aminoAcidChain.appendChild(aminoAcidEl);
                aminoAcidEl.scrollIntoView({ behavior: 'smooth', block: 'end' });
            }
            
            // Funkce pro resetování řetězce
            function resetChain() {
                aminoAcidChain.innerHTML = `
                    <div class="chain-element px-3 py-1 bg-blue-200 text-blue-800 rounded-full font-semibold">
                        <p>AUG (Met)</p>
                    </div>
                `;
                codonInput.disabled = false;
                translateBtn.classList.remove('hidden');
                resetBtn.classList.add('hidden');
                resultText.textContent = "Zadejte kodon a začněte tvořit!";
                resultText.classList.remove('text-green-700', 'text-red-700', 'text-yellow-600');
                resultText.classList.add('text-gray-700');
                explanationContainer.classList.add('hidden');
                explainBtn.disabled = true;
                continueBtn.disabled = true;
            }

            // Funkce pro překlad kodonu
            function translateCodon() {
                const codon = codonInput.value.toUpperCase();
                codonInput.value = ''; // Vyprázdní vstupní pole po zadání

                resultText.classList.remove('text-green-700', 'text-red-700', 'text-yellow-600');
                resultText.classList.add('text-gray-700');
                explanationContainer.classList.add('hidden');
                explainBtn.disabled = true;
                continueBtn.disabled = true;

                if (codon.length !== 3 || !/^[AUGC]+$/.test(codon)) {
                    resultText.textContent = "Neplatný kodon! Zadejte 3 písmena (A, U, G, C).";
                    resultText.classList.remove('text-gray-700');
                    resultText.classList.add('text-yellow-600');
                    return;
                }

                const aminoAcid = geneticCode[codon];

                if (aminoAcid) {
                    resultText.textContent = aminoAcid;
                    if (aminoAcid === "STOP") {
                        resultText.classList.remove('text-gray-700');
                        resultText.classList.add('text-red-700');
                        addAminoAcidToChain("STOP", true);
                        codonInput.disabled = true; // Zablokuje vstup
                        translateBtn.classList.add('hidden'); // Skryje tlačítko Přeložit
                        resetBtn.classList.remove('hidden'); // Zobrazí tlačítko Reset
                        explainBtn.disabled = false;
                    } else {
                        resultText.classList.remove('text-gray-700');
                        resultText.classList.add('text-green-700');
                        addAminoAcidToChain(aminoAcid, false);
                        explainBtn.disabled = false;
                        continueBtn.disabled = false;
                    }
                } else {
                    resultText.textContent = "Kodon nebyl nalezen v tabulce. Zkuste jiný.";
                    resultText.classList.remove('text-gray-700');
                    resultText.classList.add('text-yellow-600');
                }
            }

            // Funkce pro vysvětlení aminokyseliny pomocí Gemini API
            async function explainAminoAcid() {
                const chainElements = aminoAcidChain.querySelectorAll('.chain-element p');
                const lastAminoAcidEl = chainElements[chainElements.length - 1];
                const lastAminoAcidName = lastAminoAcidEl.textContent;

                if (!lastAminoAcidName || lastAminoAcidName.includes("Met")) {
                    explanationText.textContent = "Nejprve přeložte alespoň jeden kodon, abych mohl poskytnout vysvětlení.";
                    explanationContainer.classList.remove('hidden');
                    return;
                }

                explanationContainer.classList.remove('hidden');
                explanationText.innerHTML = `<span class="italic text-gray-500">Generuji vysvětlení...</span>`;

                const prompt = `Vysvětli jednoduše funkci a význam aminokyseliny ${lastAminoAcidName} v lidském těle. Odpověz stručně, maximálně ve dvou větách a použij český jazyk.`;
                let chatHistory = [];
                chatHistory.push({ role: "user", parts: [{ text: prompt }] });
                const payload = { contents: chatHistory };
                const apiKey = ""
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });
                    const result = await response.json();
                    const text = result.candidates[0].content.parts[0].text;
                    explanationText.textContent = text;
                } catch (error) {
                    explanationText.textContent = "Nepodařilo se získat vysvětlení. Zkuste to prosím znovu.";
                    console.error('Error fetching explanation:', error);
                }
            }
            
            // Funkce pro pokračování řetězce pomocí Gemini API
            async function continueChain() {
                const chainElements = aminoAcidChain.querySelectorAll('.chain-element p');
                const currentChain = Array.from(chainElements).map(el => el.textContent).join(' → ');

                continueBtn.disabled = true;
                const originalBtnText = continueBtn.textContent;
                continueBtn.textContent = 'Generuji...';

                const prompt = `Jsi genetický překladač. Vytvoř tři genetické kodony (třípísmenné sekvence, např. 'UUC', 'GCA', 'UAG') které by mohly navazovat na tento řetězec aminokyselin: "${currentChain}". Zpět vrať pouze pole JSON s klíčem 'codons' a hodnotou pole řetězců. Neposkytuj žádný jiný text ani vysvětlení, jen čistý JSON.`;
                let chatHistory = [];
                chatHistory.push({ role: "user", parts: [{ text: prompt }] });

                const payload = {
                    contents: chatHistory,
                    generationConfig: {
                        responseMimeType: "application/json",
                        responseSchema: {
                            type: "OBJECT",
                            properties: {
                                "codons": {
                                    "type": "ARRAY",
                                    "items": { "type": "STRING" }
                                }
                            }
                        }
                    }
                };

                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });
                    const result = await response.json();
                    
                    const jsonString = result.candidates[0].content.parts[0].text;
                    const parsedJson = JSON.parse(jsonString);
                    const newCodons = parsedJson.codons;

                    for (const codon of newCodons) {
                        const aminoAcid = geneticCode[codon.toUpperCase()];
                        if (aminoAcid) {
                             addAminoAcidToChain(aminoAcid, aminoAcid === "STOP");
                             if (aminoAcid === "STOP") {
                                // Stop the loop if a stop codon is generated
                                break;
                            }
                        }
                    }
                } catch (error) {
                    console.error('Error fetching new codons:', error);
                    resultText.textContent = "Nepodařilo se vygenerovat další kodony. Zkuste to prosím znovu.";
                } finally {
                    continueBtn.textContent = originalBtnText;
                    continueBtn.disabled = false;
                }
            }


            // Přiřazení událostí k prvkům
            translateBtn.addEventListener('click', translateCodon);
            codonInput.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') {
                    translateCodon();
                }
            });
            resetBtn.addEventListener('click', resetChain);
            explainBtn.addEventListener('click', explainAminoAcid);
            continueBtn.addEventListener('click', continueChain);
        });
    </script>
</body>
</html>
