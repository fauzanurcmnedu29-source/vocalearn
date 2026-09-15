<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VocaLearn - Belajar Kosakata</title>
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/xlsx/dist/xlsx.full.min.js"></script>
    <style>
        * { -webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        .line-clamp-2 { display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
        .glass-card { backdrop-filter: blur(10px); background: rgba(255, 255, 255, 0.05); border: 1px solid rgba(255, 255, 255, 0.1); }
        @keyframes shine { 0% { background-position: -200% center; } 100% { background-position: 200% center; } }
        .shine-effect { animation: shine 3s infinite; background-size: 200% 100%; background-image: linear-gradient(90deg, transparent, rgba(255,255,255,0.1), transparent); }
        @keyframes float { 0%, 100% { transform: translateY(0px); } 50% { transform: translateY(-10px); } }
        .float-animation { animation: float 3s ease-in-out infinite; }
        @keyframes toastIn { 0% { opacity: 0; transform: translateY(10px); } 100% { opacity: 1; transform: translateY(0); } }
        .toast-in { animation: toastIn 0.25s ease-out; }
        html { scroll-behavior: smooth; }
    </style>
</head>
<body class="bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        // ---- Small inline icon components (no external package dependency) ----
        const Icon = {
          Plus: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" /></svg>,
          Trash2: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>,
          BookOpen: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 6.253v13m0-13C6.5 6.253 2 10.998 2 17s4.5 10.747 10 10.747c5.5 0 10-4.998 10-10.747 0-5.002-4.5-10.747-10-10.747z" /></svg>,
          ChevronRight: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 5l7 7-7 7" /></svg>,
          Brain: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12a9 9 0 11-18 0 9 9 0 0118 0zm-5.36 5.36l-.707.707M9 19.071V20m0-6.071h.01" /></svg>,
          Target: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 10V3L4 14h7v7l9-11h-7z" /></svg>,
          Grid: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 3H5a2 2 0 00-2 2v4m0 0H3m4 0a2 2 0 012-2h4V3m6 0h4a2 2 0 012 2v4m0 0v4m0 4h-3m-3 0h-4v-4m0 0V7m0 0H9" /></svg>,
          Upload: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12" /></svg>,
          Download: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" /></svg>,
          Check: (p) => <svg {...p} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" /></svg>,
        };

        const CATEGORY_COLORS = [
          { name: 'Teal', bg: '#14B8A6' },
          { name: 'Violet', bg: '#A78BFA' },
          { name: 'Amber', bg: '#F59E0B' },
          { name: 'Pink', bg: '#EC4899' },
          { name: 'Cyan', bg: '#06B6D4' },
          { name: 'Purple', bg: '#8B5CF6' },
          { name: 'Emerald', bg: '#10B981' },
          { name: 'Rose', bg: '#F87171' },
          { name: 'Blue', bg: '#3B82F6' },
          { name: 'Indigo', bg: '#6366F1' }
        ];

        const GlassStyles = () => (
          <style>{`
            .glass-card { backdrop-filter: blur(10px); background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.1); }
          `}</style>
        );

        const VocaLearn = () => {
          const [screen, setScreen] = useState('home');
          const [vocabSets, setVocabSets] = useState(() => {
            try {
              const saved = localStorage.getItem('vocaLearnSetsV2');
              return saved ? JSON.parse(saved) : [];
            } catch (err) {
              console.error('Gagal membaca data tersimpan:', err);
              return [];
            }
          });

          const [currentSetId, setCurrentSetId] = useState(null);
          const [currentCategoryId, setCurrentCategoryId] = useState(null);
          const [newSetName, setNewSetName] = useState('');
          const [newCategoryName, setNewCategoryName] = useState('');
          const [newWord, setNewWord] = useState('');
          const [newDefinition, setNewDefinition] = useState('');
          const [currentGameType, setCurrentGameType] = useState(null);
          const [gameState, setGameState] = useState(null);
          const [categoryColor, setCategoryColor] = useState(0);
          const [showSaveToast, setShowSaveToast] = useState(false);

          const jsonFileInputRef = useRef(null);
          const csvFileInputRef = useRef(null);
          const isFirstRender = useRef(true);

          // CSV import flow state
          const [csvImportSetId, setCsvImportSetId] = useState(null);
          const [csvImportTargetCategoryId, setCsvImportTargetCategoryId] = useState('__new__');
          const [csvImportNewCategoryName, setCsvImportNewCategoryName] = useState('');
          const [csvImportRows, setCsvImportRows] = useState([]);
          const [csvImportError, setCsvImportError] = useState('');

          // Persist to localStorage on every change, and show a brief "saved" toast
          // (skipped on the very first mount, since that's just loading existing data, not a new save)
          useEffect(() => {
            try {
              localStorage.setItem('vocaLearnSetsV2', JSON.stringify(vocabSets));
            } catch (err) {
              console.error('Gagal menyimpan data:', err);
            }
            if (isFirstRender.current) {
              isFirstRender.current = false;
              return;
            }
            setShowSaveToast(true);
            const timer = setTimeout(() => setShowSaveToast(false), 1600);
            return () => clearTimeout(timer);
          }, [vocabSets]);

          // ==================== SET / CATEGORY / VOCAB CRUD (all immutable) ====================

          const handleCreateSet = () => {
            if (!newSetName.trim()) { alert('Nama set tidak boleh kosong!'); return; }
            const newSet = {
              id: Date.now(),
              name: newSetName.trim(),
              categories: [],
              createdAt: new Date().toLocaleDateString('id-ID'),
              stats: { totalAttempts: 0, correctAnswers: 0 }
            };
            setVocabSets(prev => [...prev, newSet]);
            setNewSetName('');
            setCurrentSetId(newSet.id);
            setScreen('manage-categories');
          };

          const handleCreateCategory = (setId) => {
            if (!newCategoryName.trim()) { alert('Nama kategori tidak boleh kosong!'); return; }
            const newCategory = {
              id: Date.now(),
              name: newCategoryName.trim(),
              vocabulary: [],
              colorIndex: categoryColor,
              color: CATEGORY_COLORS[categoryColor]
            };
            setVocabSets(prev => prev.map(s =>
              s.id === setId ? { ...s, categories: [...s.categories, newCategory] } : s
            ));
            setNewCategoryName('');
            setCategoryColor((categoryColor + 1) % CATEGORY_COLORS.length);
          };

          const handleAddVocab = (setId, categoryId) => {
            if (!newWord.trim() || !newDefinition.trim()) { alert('Kata dan definisi tidak boleh kosong!'); return; }
            if (newWord.length > 100 || newDefinition.length > 500) { alert('Kata maksimal 100 karakter, definisi maksimal 500 karakter'); return; }

            const targetSet = vocabSets.find(s => s.id === setId);
            const targetCategory = targetSet?.categories.find(c => c.id === categoryId);
            if (targetCategory && targetCategory.vocabulary.length >= 1500) {
              alert('Maksimal 1500 kosakata per kategori!');
              return;
            }

            const newItem = { id: Date.now(), word: newWord.trim(), definition: newDefinition.trim() };
            setVocabSets(prev => prev.map(s => {
              if (s.id !== setId) return s;
              return {
                ...s,
                categories: s.categories.map(c =>
                  c.id === categoryId ? { ...c, vocabulary: [...c.vocabulary, newItem] } : c
                )
              };
            }));
            setNewWord('');
            setNewDefinition('');
          };

          const handleClearVocabForm = () => {
            setNewWord('');
            setNewDefinition('');
          };

          const handleDeleteVocab = (setId, categoryId, vocabId) => {
            setVocabSets(prev => prev.map(s => {
              if (s.id !== setId) return s;
              return {
                ...s,
                categories: s.categories.map(c =>
                  c.id === categoryId ? { ...c, vocabulary: c.vocabulary.filter(v => v.id !== vocabId) } : c
                )
              };
            }));
          };

          const handleDeleteCategory = (setId, categoryId) => {
            if (window.confirm('Yakin hapus kategori ini beserta semua kosakata di dalamnya? Tindakan ini tidak bisa dibatalkan.')) {
              setVocabSets(prev => prev.map(s =>
                s.id === setId ? { ...s, categories: s.categories.filter(c => c.id !== categoryId) } : s
              ));
            }
          };

          const handleDeleteSet = (setId) => {
            if (window.confirm('Yakin hapus set ini beserta semua kategori dan kosakata di dalamnya? Tindakan ini tidak bisa dibatalkan.')) {
              setVocabSets(prev => prev.filter(s => s.id !== setId));
            }
          };

          // ==================== JSON BACKUP (export / import all data) ====================

          const handleExportData = () => {
            if (vocabSets.length === 0) { alert('Belum ada data untuk di-backup.'); return; }
            const dataStr = JSON.stringify(vocabSets, null, 2);
            const blob = new Blob([dataStr], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            const timestamp = new Date().toISOString().slice(0, 10);
            a.href = url;
            a.download = `vocalearn-backup-${timestamp}.json`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
          };

          const normalizeImportedSet = (s) => ({
            id: s.id || (Date.now() + Math.floor(Math.random() * 1000)),
            name: s.name || 'Set Tanpa Nama',
            categories: Array.isArray(s.categories) ? s.categories.map(c => ({
              id: c.id || (Date.now() + Math.floor(Math.random() * 1000)),
              name: c.name || 'Kategori',
              vocabulary: Array.isArray(c.vocabulary) ? c.vocabulary : [],
              colorIndex: typeof c.colorIndex === 'number' ? c.colorIndex : 0,
              color: c.color || CATEGORY_COLORS[0]
            })) : [],
            createdAt: s.createdAt || new Date().toLocaleDateString('id-ID'),
            stats: (s.stats && typeof s.stats.totalAttempts === 'number')
              ? s.stats
              : { totalAttempts: 0, correctAnswers: 0 }
          });

          const handleImportJsonFile = (e) => {
            const file = e.target.files[0];
            e.target.value = '';
            if (!file) return;
            const reader = new FileReader();
            reader.onload = (event) => {
              try {
                const imported = JSON.parse(event.target.result);
                if (!Array.isArray(imported)) throw new Error('File ini bukan format backup VocaLearn yang valid.');
                const normalized = imported.map(normalizeImportedSet);
                const proceed = window.confirm(
                  `File ini berisi ${normalized.length} set kosakata. Import akan digabung dengan data yang sekarang ada (set dengan ID sama akan ditimpa oleh versi dari file). Lanjutkan?`
                );
                if (!proceed) return;

                setVocabSets(prev => {
                  const merged = [...prev];
                  normalized.forEach(importedSet => {
                    const idx = merged.findIndex(s => s.id === importedSet.id);
                    if (idx >= 0) merged[idx] = importedSet;
                    else merged.push(importedSet);
                  });
                  return merged;
                });
                alert('✓ Backup berhasil di-import!');
              } catch (err) {
                alert('Gagal membaca file backup: ' + err.message);
              }
            };
            reader.readAsText(file);
          };

          // ==================== CSV IMPORT (bulk-add vocabulary from spreadsheet) ====================

          // Minimal CSV parser handling quoted fields with embedded commas
          // (e.g. "hello, world","a greeting") - covers common Excel/Sheets CSV exports.
          const parseCsvText = (text) => {
            const rows = [];
            let row = [];
            let field = '';
            let inQuotes = false;

            for (let i = 0; i < text.length; i++) {
              const char = text[i];
              const next = text[i + 1];
              if (inQuotes) {
                if (char === '"' && next === '"') { field += '"'; i++; }
                else if (char === '"') { inQuotes = false; }
                else { field += char; }
              } else {
                if (char === '"') { inQuotes = true; }
                else if (char === ',') { row.push(field); field = ''; }
                else if (char === '\r') { /* skip, \n handles the line break */ }
                else if (char === '\n') { row.push(field); rows.push(row); row = []; field = ''; }
                else { field += char; }
              }
            }
            if (field.length > 0 || row.length > 0) { row.push(field); rows.push(row); }
            return rows.filter(r => r.some(cell => cell.trim() !== ''));
          };

          // Shared row-processing logic used by both CSV and Excel import paths:
          // header-row detection, trimming, 1500-cap check, and duplicate detection.
          // String(cell ?? '') is used instead of (cell || '') because Excel cells
          // can come back as numbers (not just strings) from SheetJS.
          const processImportedRows = (rawRows) => {
            if (rawRows.length === 0) { setCsvImportError('File kosong atau tidak terbaca.'); return; }

            let dataRows = rawRows;
            const firstRow = rawRows[0].map(c => String(c ?? '').trim().toLowerCase());
            const looksLikeHeader =
              (firstRow[0] === 'kata' || firstRow[0] === 'word') &&
              (firstRow[1] === 'definisi' || firstRow[1] === 'definition' || firstRow[1] === 'arti');
            if (looksLikeHeader) dataRows = rawRows.slice(1);

            const parsedRows = dataRows
              .map(r => ({ word: String(r[0] ?? '').trim(), definition: String(r[1] ?? '').trim() }))
              .filter(r => r.word !== '' && r.definition !== '');

            if (parsedRows.length === 0) {
              setCsvImportError('Tidak ada baris valid ditemukan. Pastikan file punya 2 kolom: kata dan definisi.');
              return;
            }
            if (parsedRows.length > 1500) {
              setCsvImportError(`File berisi ${parsedRows.length} baris, melebihi batas 1500 kosakata per kategori. Pisahkan jadi beberapa file/kategori.`);
              return;
            }

            const targetSet = vocabSets.find(s => s.id === csvImportSetId);
            const targetCategory = csvImportTargetCategoryId !== '__new__'
              ? targetSet?.categories.find(c => c.id === csvImportTargetCategoryId)
              : null;
            const existingWordsLower = new Set(
              (targetCategory?.vocabulary || []).map(v => v.word.trim().toLowerCase())
            );

            const seenInFile = new Set();
            const rowsWithFlags = parsedRows.map(r => {
              const key = r.word.toLowerCase();
              const isDuplicate = existingWordsLower.has(key) || seenInFile.has(key);
              seenInFile.add(key);
              return { ...r, isDuplicate, include: !isDuplicate };
            });

            setCsvImportRows(rowsWithFlags);
          };

          const handleSpreadsheetFileSelected = (e) => {
            const file = e.target.files[0];
            e.target.value = '';
            if (!file) return;

            setCsvImportError('');
            const fileName = file.name.toLowerCase();
            const isExcel = fileName.endsWith('.xlsx') || fileName.endsWith('.xls');

            if (isExcel) {
              if (typeof XLSX === 'undefined') {
                setCsvImportError('Pustaka pembaca Excel gagal dimuat (kemungkinan tidak ada koneksi internet). Coba lagi, atau gunakan file CSV sebagai alternatif.');
                return;
              }
              const reader = new FileReader();
              reader.onload = (event) => {
                try {
                  const data = new Uint8Array(event.target.result);
                  const workbook = XLSX.read(data, { type: 'array' });
                  const firstSheetName = workbook.SheetNames[0];
                  const worksheet = workbook.Sheets[firstSheetName];
                  const rawRows = XLSX.utils.sheet_to_json(worksheet, { header: 1, defval: '' });
                  processImportedRows(rawRows);
                } catch (err) {
                  setCsvImportError('Gagal membaca file Excel: ' + err.message + '. Pastikan file tidak rusak dan berformat .xlsx atau .xls.');
                }
              };
              reader.onerror = () => setCsvImportError('Gagal membaca file.');
              reader.readAsArrayBuffer(file);
            } else {
              const reader = new FileReader();
              reader.onload = (event) => {
                try {
                  const rawRows = parseCsvText(event.target.result);
                  processImportedRows(rawRows);
                } catch (err) {
                  setCsvImportError('Gagal membaca file: ' + err.message);
                }
              };
              reader.onerror = () => setCsvImportError('Gagal membaca file.');
              reader.readAsText(file);
            }
          };

          const toggleCsvRowInclude = (idx) => {
            setCsvImportRows(prev => prev.map((r, i) => i === idx ? { ...r, include: !r.include } : r));
          };

          const handleConfirmCsvImport = () => {
            const rowsToImport = csvImportRows.filter(r => r.include);
            if (rowsToImport.length === 0) { alert('Tidak ada kosakata yang dipilih untuk diimpor.'); return; }

            if (csvImportTargetCategoryId === '__new__') {
              if (!csvImportNewCategoryName.trim()) { alert('Beri nama untuk kategori baru terlebih dahulu.'); return; }
              const newCategory = {
                id: Date.now(),
                name: csvImportNewCategoryName.trim(),
                vocabulary: rowsToImport.map((r, i) => ({ id: Date.now() + i, word: r.word, definition: r.definition })),
                colorIndex: categoryColor,
                color: CATEGORY_COLORS[categoryColor]
              };
              setVocabSets(prev => prev.map(s =>
                s.id === csvImportSetId ? { ...s, categories: [...s.categories, newCategory] } : s
              ));
              setCategoryColor((categoryColor + 1) % CATEGORY_COLORS.length);
            } else {
              setVocabSets(prev => prev.map(s => {
                if (s.id !== csvImportSetId) return s;
                return {
                  ...s,
                  categories: s.categories.map(c => {
                    if (c.id !== csvImportTargetCategoryId) return c;
                    const room = 1500 - c.vocabulary.length;
                    const capped = rowsToImport.slice(0, room);
                    return {
                      ...c,
                      vocabulary: [...c.vocabulary, ...capped.map((r, i) => ({ id: Date.now() + i, word: r.word, definition: r.definition }))]
                    };
                  })
                };
              }));
            }

            alert(`✓ ${rowsToImport.length} kosakata berhasil diimpor!`);
            setCsvImportRows([]);
            setCsvImportError('');
            setCsvImportNewCategoryName('');
            setScreen('manage-categories');
          };

          // ==================== GAMES ====================

          const handleStartGame = (gameType) => {
            const set = vocabSets.find(s => s.id === currentSetId);
            const category = set.categories.find(c => c.id === currentCategoryId);
            if (category.vocabulary.length === 0) { alert('Tambahkan kosakata dulu!'); return; }
            if ((gameType === 'quiz' || gameType === 'match') && category.vocabulary.length < 4) {
              const proceed = window.confirm(
                `Kategori ini baru punya ${category.vocabulary.length} kosakata. Game pilihan ganda idealnya butuh minimal 4 supaya ada 4 opsi jawaban. Lanjut main dengan opsi yang lebih sedikit?`
              );
              if (!proceed) return;
            }
            setCurrentGameType(gameType);
            initializeGame(gameType, category);
            setScreen('practice');
          };

          const initializeGame = (gameType, category) => {
            const shuffled = [...category.vocabulary].sort(() => Math.random() - 0.5);

            if (gameType === 'flashcard') {
              setGameState({
                currentIndex: 0, isFlipped: false, vocabulary: shuffled, score: 0,
                total: Math.min(shuffled.length, 15), category: category.name
              });
            } else if (gameType === 'quiz') {
              setGameState({
                currentIndex: 0, vocabulary: shuffled, score: 0, total: Math.min(shuffled.length, 15),
                answered: false, isCorrect: false, selectedAnswer: null, category: category.name
              });
            } else if (gameType === 'match') {
              setGameState({
                currentIndex: 0, vocabulary: shuffled, score: 0, total: Math.min(shuffled.length, 15),
                answered: false, isCorrect: false, category: category.name
              });
            } else if (gameType === 'memory') {
              const memVocab = shuffled.slice(0, Math.min(8, shuffled.length));
              const memCards = [];
              memVocab.forEach(v => {
                // IMPORTANT: both cards share the SAME id (v.id) so the match-check
                // (card1.id === card2.id) can actually succeed. `type` distinguishes
                // the word card from the definition card for rendering/safety checks.
                memCards.push({ id: v.id, content: v.word, type: 'word' });
                memCards.push({ id: v.id, content: v.definition, type: 'definition' });
              });
              setGameState({
                cards: memCards.sort(() => Math.random() - 0.5),
                flipped: [], matched: [], moves: 0, score: 0, category: category.name
              });
            }
          };

          const endGame = (finalScore) => {
            setVocabSets(prev => prev.map(s =>
              s.id === currentSetId
                ? { ...s, stats: { totalAttempts: s.stats.totalAttempts + 1, correctAnswers: s.stats.correctAnswers + finalScore } }
                : s
            ));
            setGameState(prev => ({ ...prev, finished: true, finalScore }));
          };

          // ---- Game UI components ----

          const FlashcardGame = () => {
            const current = gameState.vocabulary[gameState.currentIndex];
            const progress = gameState.currentIndex + 1;
            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6 flex flex-col items-center justify-center">
                <div className="w-full max-w-2xl">
                  <div className="mb-8">
                    <div className="flex justify-between items-center mb-4">
                      <div>
                        <p className="text-sm text-slate-400">Kartu Flash</p>
                        <p className="text-2xl font-bold text-white">{gameState.category}</p>
                      </div>
                      <div className="text-right">
                        <p className="text-sm text-slate-400">Progress</p>
                        <p className="text-2xl font-bold text-white">{progress}/{gameState.total}</p>
                      </div>
                    </div>
                    <svg className="w-full h-2 mt-4" viewBox="0 0 100 4">
                      <rect x="0" y="0" width="100" height="4" rx="2" fill="#374151" />
                      <rect x="0" y="0" width={`${(progress / gameState.total) * 100}`} height="4" rx="2" fill="#14B8A6" />
                    </svg>
                  </div>
                  <div
                    onClick={() => setGameState({ ...gameState, isFlipped: !gameState.isFlipped })}
                    className="glass-card rounded-3xl p-16 min-h-96 flex flex-col items-center justify-center cursor-pointer transform transition-all duration-300 hover:shadow-2xl hover:shadow-teal-500/20 group relative overflow-hidden"
                  >
                    <div className="shine-effect absolute inset-0 opacity-0 group-hover:opacity-100 transition-opacity" />
                    <div className="relative z-10 text-center">
                      <p className="text-sm text-slate-400 mb-6">{gameState.isFlipped ? 'Definisi' : 'Kata'}</p>
                      <p className="text-5xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-teal-400 via-cyan-400 to-blue-400">
                        {gameState.isFlipped ? current.definition : current.word}
                      </p>
                    </div>
                    <p className="mt-12 text-sm text-slate-500">Klik untuk flip kartu</p>
                  </div>
                  <div className="mt-8 flex gap-4 justify-center">
                    <button
                      onClick={() => { if (gameState.currentIndex > 0) setGameState({ ...gameState, currentIndex: gameState.currentIndex - 1, isFlipped: false }); }}
                      disabled={gameState.currentIndex === 0}
                      className="px-8 py-3 rounded-xl bg-slate-700 text-white disabled:opacity-30 hover:bg-slate-600 transition-colors font-medium"
                    >
                      ← Prev
                    </button>
                    <button
                      onClick={() => {
                        if (gameState.currentIndex < gameState.total - 1) {
                          setGameState({ ...gameState, currentIndex: gameState.currentIndex + 1, isFlipped: false });
                        } else {
                          endGame(gameState.score);
                        }
                      }}
                      className="px-8 py-3 rounded-xl bg-gradient-to-r from-teal-500 to-cyan-500 text-white hover:shadow-lg hover:shadow-teal-500/50 transition-all font-medium"
                    >
                      {gameState.currentIndex === gameState.total - 1 ? 'Finish' : 'Next →'}
                    </button>
                  </div>
                </div>
              </div>
            );
          };

          const QuizGame = () => {
            const current = gameState.vocabulary[gameState.currentIndex];
            const distractors = gameState.vocabulary
              .filter(v => v.id !== current.id)
              .sort(() => Math.random() - 0.5)
              .slice(0, 3)
              .map(v => v.definition);
            const allOptions = [current.definition, ...distractors].sort(() => Math.random() - 0.5);
            const progress = gameState.currentIndex + 1;

            const handleAnswer = (answer) => {
              const isCorrect = answer === current.definition;
              setGameState({ ...gameState, answered: true, isCorrect, selectedAnswer: answer, score: isCorrect ? gameState.score + 1 : gameState.score });
            };

            const handleNext = () => {
              if (gameState.currentIndex < gameState.total - 1) {
                setGameState({ ...gameState, currentIndex: gameState.currentIndex + 1, answered: false, isCorrect: false, selectedAnswer: null });
              } else {
                endGame(gameState.score);
              }
            };

            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6 flex flex-col items-center justify-center">
                <div className="w-full max-w-2xl">
                  <div className="mb-8">
                    <div className="flex justify-between items-center mb-4">
                      <div>
                        <p className="text-sm text-slate-400">Quiz Kosakata</p>
                        <p className="text-xl font-bold text-white">Score: {gameState.score}/{gameState.total}</p>
                      </div>
                      <div className="text-right"><p className="text-sm text-slate-400">{progress}/{gameState.total}</p></div>
                    </div>
                    <svg className="w-full h-2 mt-4" viewBox="0 0 100 4">
                      <rect x="0" y="0" width="100" height="4" rx="2" fill="#374151" />
                      <rect x="0" y="0" width={`${(progress / gameState.total) * 100}`} height="4" rx="2" fill="#F59E0B" />
                    </svg>
                  </div>
                  <div className="glass-card rounded-3xl p-10 mb-8">
                    <p className="text-slate-400 text-center mb-4">Apa definisi dari:</p>
                    <p className="text-4xl font-bold text-center text-transparent bg-clip-text bg-gradient-to-r from-amber-400 to-orange-400">{current.word}</p>
                  </div>
                  <div className="space-y-3 mb-8">
                    {allOptions.map((option, idx) => (
                      <button
                        key={idx}
                        onClick={() => !gameState.answered && handleAnswer(option)}
                        disabled={gameState.answered}
                        className={`w-full p-4 rounded-xl transition-all duration-300 font-medium ${
                          !gameState.answered
                            ? 'glass-card hover:border-amber-500/50 cursor-pointer hover:shadow-lg hover:shadow-amber-500/10'
                            : option === current.definition
                            ? 'bg-emerald-500/20 border border-emerald-500/50 text-emerald-300'
                            : option === gameState.selectedAnswer
                            ? 'bg-rose-500/20 border border-rose-500/50 text-rose-300'
                            : 'glass-card opacity-50'
                        }`}
                      >
                        <span className="text-white">{option}</span>
                      </button>
                    ))}
                  </div>
                  {gameState.answered && (
                    <div>
                      <div className={`mb-6 p-4 rounded-xl text-center font-bold ${gameState.isCorrect ? 'bg-emerald-500/20 text-emerald-300 border border-emerald-500/30' : 'bg-rose-500/20 text-rose-300 border border-rose-500/30'}`}>
                        {gameState.isCorrect ? '✓ Benar!' : '✗ Salah!'}
                      </div>
                      <button onClick={handleNext} className="w-full py-3 rounded-xl bg-gradient-to-r from-amber-500 to-orange-500 text-white font-bold hover:shadow-lg hover:shadow-amber-500/50 transition-all">
                        {gameState.currentIndex === gameState.total - 1 ? 'Lihat Hasil' : 'Lanjut →'}
                      </button>
                    </div>
                  )}
                </div>
              </div>
            );
          };

          const MatchGame = () => {
            const current = gameState.vocabulary[gameState.currentIndex];
            const correctAnswer = current.word;
            const distractors = gameState.vocabulary
              .filter(v => v.id !== current.id)
              .sort(() => Math.random() - 0.5)
              .slice(0, 3)
              .map(v => v.word);
            const options = [current.word, ...distractors].sort(() => Math.random() - 0.5);
            const progress = gameState.currentIndex + 1;

            const handleAnswer = (answer) => {
              const isCorrect = answer === correctAnswer;
              setGameState({ ...gameState, answered: true, isCorrect, score: isCorrect ? gameState.score + 1 : gameState.score });
            };

            const handleNext = () => {
              if (gameState.currentIndex < gameState.total - 1) {
                setGameState({ ...gameState, currentIndex: gameState.currentIndex + 1, answered: false });
              } else {
                endGame(gameState.score);
              }
            };

            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6 flex flex-col items-center justify-center">
                <div className="w-full max-w-2xl">
                  <div className="mb-8">
                    <div className="flex justify-between items-center mb-4">
                      <div>
                        <p className="text-sm text-slate-400">Catur Jawa</p>
                        <p className="text-xl font-bold text-white">Score: {gameState.score}/{gameState.total}</p>
                      </div>
                      <div className="text-right"><p className="text-sm text-slate-400">{progress}/{gameState.total}</p></div>
                    </div>
                    <svg className="w-full h-2 mt-4" viewBox="0 0 100 4">
                      <rect x="0" y="0" width="100" height="4" rx="2" fill="#374151" />
                      <rect x="0" y="0" width={`${(progress / gameState.total) * 100}`} height="4" rx="2" fill="#F87171" />
                    </svg>
                  </div>
                  <div className="glass-card rounded-3xl p-10 mb-8">
                    <p className="text-slate-400 text-center mb-4">Pilih kata yang artinya:</p>
                    <p className="text-4xl font-bold text-center text-transparent bg-clip-text bg-gradient-to-r from-rose-400 to-pink-400">{current.definition}</p>
                  </div>
                  <div className="grid grid-cols-2 gap-4 mb-8">
                    {options.map((option, idx) => (
                      <button
                        key={idx}
                        onClick={() => !gameState.answered && handleAnswer(option)}
                        disabled={gameState.answered}
                        className={`p-4 rounded-xl transition-all duration-300 font-bold ${
                          !gameState.answered
                            ? 'glass-card hover:border-rose-500/50 cursor-pointer hover:shadow-lg hover:shadow-rose-500/10'
                            : option === correctAnswer
                            ? 'bg-emerald-500/20 border border-emerald-500/50 text-emerald-300'
                            : 'glass-card opacity-50'
                        }`}
                      >
                        <span className="text-white text-sm">{option}</span>
                      </button>
                    ))}
                  </div>
                  {gameState.answered && (
                    <div>
                      <div className={`mb-6 p-4 rounded-xl text-center font-bold ${gameState.isCorrect ? 'bg-emerald-500/20 text-emerald-300 border border-emerald-500/30' : 'bg-rose-500/20 text-rose-300 border border-rose-500/30'}`}>
                        {gameState.isCorrect ? '✓ Benar!' : '✗ Salah!'}
                      </div>
                      <button onClick={handleNext} className="w-full py-3 rounded-xl bg-gradient-to-r from-rose-500 to-pink-500 text-white font-bold hover:shadow-lg hover:shadow-rose-500/50 transition-all">
                        {gameState.currentIndex === gameState.total - 1 ? 'Lihat Hasil' : 'Lanjut →'}
                      </button>
                    </div>
                  )}
                </div>
              </div>
            );
          };

          const MemoryGame = () => {
            const handleCardClick = (idx) => {
              if (gameState.flipped.length >= 2 || gameState.flipped.includes(idx) || gameState.matched.includes(gameState.cards[idx].id)) return;

              const newFlipped = [...gameState.flipped, idx];
              setGameState({ ...gameState, flipped: newFlipped });

              if (newFlipped.length === 2) {
                const [idx1, idx2] = newFlipped;
                const card1 = gameState.cards[idx1];
                const card2 = gameState.cards[idx2];

                // Fixed: word card and definition card now share the same id,
                // and the type check ensures a word never "matches" another word.
                if (card1.id === card2.id && card1.type !== card2.type) {
                  setGameState({
                    ...gameState,
                    matched: [...gameState.matched, card1.id],
                    flipped: [],
                    moves: gameState.moves + 1,
                    score: gameState.score + 10
                  });
                } else {
                  setTimeout(() => {
                    setGameState(prev => ({ ...prev, flipped: [], moves: prev.moves + 1 }));
                  }, 800);
                }
              }
            };

            const isComplete = gameState.matched.length === gameState.cards.length / 2;

            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6 flex flex-col items-center justify-center">
                <div className="w-full max-w-2xl">
                  <div className="mb-8 grid grid-cols-2 gap-4">
                    <div className="glass-card rounded-xl p-6 text-center">
                      <p className="text-sm text-slate-400 mb-2">Poin</p>
                      <p className="text-3xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-teal-400 to-cyan-400">{gameState.score}</p>
                    </div>
                    <div className="glass-card rounded-xl p-6 text-center">
                      <p className="text-sm text-slate-400 mb-2">Gerakan</p>
                      <p className="text-3xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-pink-400">{gameState.moves}</p>
                    </div>
                  </div>
                  <div className="grid grid-cols-4 gap-2 mb-8">
                    {gameState.cards.map((card, idx) => {
                      const isMatched = gameState.matched.includes(card.id);
                      const isFlippedNow = gameState.flipped.includes(idx);
                      return (
                        <button
                          key={idx}
                          onClick={() => handleCardClick(idx)}
                          disabled={isMatched || isFlippedNow}
                          className={`aspect-square rounded-xl font-bold text-xs p-2 text-center flex items-center justify-center transition-all transform ${
                            isMatched
                              ? 'bg-emerald-500/20 border border-emerald-500/50 cursor-default'
                              : isFlippedNow
                              ? 'glass-card border-purple-500/50 scale-95'
                              : 'glass-card hover:border-purple-500/50 cursor-pointer hover:shadow-lg hover:shadow-purple-500/10'
                          }`}
                        >
                          {isFlippedNow || isMatched ? (
                            <span className="line-clamp-2 text-white">{card.content}</span>
                          ) : (
                            <span className="text-slate-400 text-lg">?</span>
                          )}
                        </button>
                      );
                    })}
                  </div>
                  {isComplete && (
                    <div className="glass-card rounded-2xl p-8 text-center mb-6 border border-emerald-500/30">
                      <p className="text-3xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-emerald-400 to-teal-400 mb-2">🎉 Sempurna!</p>
                      <p className="text-slate-300">Poin akhir: <span className="font-bold text-2xl text-emerald-400">{gameState.score}</span></p>
                    </div>
                  )}
                  <button
                    onClick={() => {
                      if (isComplete) endGame(gameState.score);
                      else { setCurrentGameType(null); setScreen('games'); }
                    }}
                    className="w-full py-3 rounded-xl bg-gradient-to-r from-purple-500 to-pink-500 text-white font-bold hover:shadow-lg hover:shadow-purple-500/50 transition-all"
                  >
                    {isComplete ? 'Selesai' : 'Keluar'}
                  </button>
                </div>
              </div>
            );
          };

          // ==================== SCREENS ====================

          if (screen === 'home') {
            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 container mx-auto px-4 py-16">
                <div className="text-center mb-16">
                  <div className="inline-block mb-6 float-animation text-5xl">✨</div>
                  <h1 className="text-6xl font-black mb-4">
                    <span className="bg-gradient-to-r from-teal-400 via-cyan-400 to-blue-400 bg-clip-text text-transparent">VocaLearn</span>
                  </h1>
                  <p className="text-xl text-slate-300 mb-2">Belajar kosakata dengan cara yang menyenangkan</p>
                  <p className="text-slate-400">Kelompokkan, mainkan, dan kuasai</p>
                </div>

                <button
                  onClick={() => { setNewSetName(''); setScreen('create-set'); }}
                  className="w-full mb-4 py-4 px-6 bg-gradient-to-r from-teal-500 to-cyan-500 text-white rounded-2xl font-bold text-lg hover:shadow-2xl hover:shadow-teal-500/30 transition-all transform hover:scale-105 flex items-center justify-center gap-3"
                >
                  <Icon.Plus className="w-6 h-6" />
                  Buat Set Kosakata Baru
                </button>

                <div className="flex gap-3 mb-12">
                  <input type="file" accept=".json,application/json" ref={jsonFileInputRef} onChange={handleImportJsonFile} className="hidden" />
                  <button
                    onClick={handleExportData}
                    className="flex-1 py-2 px-4 rounded-xl glass-card text-slate-300 text-sm font-medium hover:text-white hover:border-slate-500 transition-colors flex items-center justify-center gap-2"
                  >
                    <Icon.Download className="w-4 h-4" /> Cadangkan Data
                  </button>
                  <button
                    onClick={() => jsonFileInputRef.current?.click()}
                    className="flex-1 py-2 px-4 rounded-xl glass-card text-slate-300 text-sm font-medium hover:text-white hover:border-slate-500 transition-colors flex items-center justify-center gap-2"
                  >
                    <Icon.Upload className="w-4 h-4" /> Pulihkan Data
                  </button>
                </div>

                {vocabSets.length === 0 ? (
                  <div className="glass-card rounded-2xl p-12 text-center">
                    <Icon.BookOpen className="w-16 h-16 text-slate-500 mx-auto mb-4" />
                    <p className="text-slate-400">Belum ada set. Buat yang pertama sekarang!</p>
                  </div>
                ) : (
                  <div className="grid gap-4">
                    {vocabSets.map(set => {
                      const totalVocab = set.categories.reduce((sum, cat) => sum + cat.vocabulary.length, 0);
                      return (
                        <div key={set.id} className="glass-card rounded-2xl p-6 hover:shadow-xl hover:shadow-teal-500/10 hover:border-teal-500/30 transition-all group border border-slate-700">
                          <div className="flex justify-between items-start mb-4">
                            <div className="flex-1">
                              <h3 className="text-2xl font-bold text-white group-hover:text-teal-400 transition-colors">{set.name}</h3>
                              <p className="text-sm text-slate-400 mt-1">{set.categories.length} kategori • {totalVocab} kosakata</p>
                            </div>
                            <button onClick={() => handleDeleteSet(set.id)} className="p-2 hover:bg-rose-500/20 rounded-lg text-rose-400 transition-colors">
                              <Icon.Trash2 className="w-5 h-5" />
                            </button>
                          </div>
                          <div className="grid grid-cols-2 gap-3 mb-4">
                            <div className="bg-teal-500/10 rounded-lg p-3 border border-teal-500/20">
                              <p className="text-xs text-slate-400">Total Coba</p>
                              <p className="text-2xl font-bold text-teal-400">{set.stats.totalAttempts}</p>
                            </div>
                            <div className="bg-cyan-500/10 rounded-lg p-3 border border-cyan-500/20">
                              <p className="text-xs text-slate-400">Jawaban Benar</p>
                              <p className="text-2xl font-bold text-cyan-400">{set.stats.correctAnswers}</p>
                            </div>
                          </div>
                          <div className="flex gap-3">
                            <button
                              onClick={() => { setCurrentSetId(set.id); setScreen('manage-categories'); }}
                              className="flex-1 py-2 px-4 bg-slate-700/50 text-white rounded-lg font-semibold hover:bg-slate-600 transition-colors"
                            >
                              Kelola
                            </button>
                            <button
                              onClick={() => {
                                setCurrentSetId(set.id);
                                if (set.categories.length === 0) { alert('Buat kategori dulu!'); setScreen('manage-categories'); }
                                else setScreen('select-category');
                              }}
                              className="flex-1 py-2 px-4 bg-gradient-to-r from-teal-500 to-cyan-500 text-white rounded-lg font-semibold hover:shadow-lg hover:shadow-teal-500/30 transition-all flex items-center justify-center gap-2"
                            >
                              Main <Icon.ChevronRight className="w-4 h-4" />
                            </button>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                )}

                {showSaveToast && (
                  <div className="fixed bottom-6 right-6 toast-in">
                    <div className="glass-card rounded-xl px-5 py-3 flex items-center gap-2 border border-emerald-500/40 shadow-2xl">
                      <Icon.Check className="w-4 h-4 text-emerald-400" />
                      <span className="text-sm font-medium text-emerald-300">Tersimpan</span>
                    </div>
                  </div>
                )}
              </div>
            );
          }

          if (screen === 'create-set') {
            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6 flex items-center justify-center">
                <div className="w-full max-w-md">
                  <div className="glass-card rounded-3xl p-8">
                    <h2 className="text-3xl font-bold text-white mb-6">Buat Set Baru</h2>
                    <input
                      type="text"
                      value={newSetName}
                      onChange={(e) => setNewSetName(e.target.value)}
                      placeholder="Nama set (cth: Bahasa Inggris)"
                      className="w-full px-6 py-4 rounded-xl bg-slate-800/50 border border-slate-600 text-white placeholder-slate-500 focus:border-teal-500 focus:outline-none mb-6 transition-colors"
                    />
                    <div className="flex gap-3">
                      <button onClick={() => setScreen('home')} className="flex-1 py-3 rounded-xl bg-slate-700 text-white font-bold hover:bg-slate-600 transition-colors">Batal</button>
                      <button onClick={handleCreateSet} className="flex-1 py-3 rounded-xl bg-gradient-to-r from-teal-500 to-cyan-500 text-white font-bold hover:shadow-lg hover:shadow-teal-500/50 transition-all">Lanjut</button>
                    </div>
                  </div>
                </div>
              </div>
            );
          }

          if (screen === 'manage-categories') {
            const set = vocabSets.find(s => s.id === currentSetId);
            if (!set) return null;
            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6">
                <div className="max-w-4xl mx-auto">
                  <div className="flex justify-between items-center mb-8">
                    <div>
                      <h2 className="text-4xl font-bold text-white">{set.name}</h2>
                      <p className="text-slate-400 mt-1">{set.categories.length} kategori</p>
                    </div>
                    <button onClick={() => setScreen('home')} className="px-6 py-3 bg-slate-700 text-white rounded-lg font-semibold hover:bg-slate-600 transition-colors">← Kembali</button>
                  </div>

                  <div className="glass-card rounded-2xl p-8 mb-6">
                    <h3 className="text-2xl font-bold text-white mb-6">Buat Kategori Baru</h3>
                    <input
                      type="text"
                      value={newCategoryName}
                      onChange={(e) => setNewCategoryName(e.target.value)}
                      placeholder="Nama kategori (cth: Minggu 1)"
                      className="w-full px-6 py-4 rounded-xl bg-slate-800/50 border border-slate-600 text-white placeholder-slate-500 focus:border-teal-500 focus:outline-none mb-4 transition-colors"
                    />
                    <div className="mb-4">
                      <p className="text-sm text-slate-400 mb-3">Pilih warna:</p>
                      <div className="grid grid-cols-5 gap-3">
                        {CATEGORY_COLORS.map((color, idx) => (
                          <button
                            key={idx}
                            onClick={() => setCategoryColor(idx)}
                            className={`w-full aspect-square rounded-lg transition-all ${categoryColor === idx ? 'ring-2 ring-offset-2 ring-offset-slate-900 ring-white scale-110' : ''}`}
                            style={{ backgroundColor: color.bg }}
                          />
                        ))}
                      </div>
                    </div>
                    <button onClick={() => handleCreateCategory(set.id)} className="w-full py-3 rounded-xl bg-gradient-to-r from-teal-500 to-cyan-500 text-white font-bold hover:shadow-lg hover:shadow-teal-500/50 transition-all">
                      + Buat Kategori
                    </button>
                  </div>

                  <div className="glass-card rounded-2xl p-8 mb-8 border border-emerald-500/20">
                    <h3 className="text-2xl font-bold text-white">Import dari CSV/Excel</h3>
                    <p className="text-sm text-slate-400 mt-1">Tambah banyak kosakata sekaligus dari file spreadsheet, tanpa isi satu-satu.</p>
                    <button
                      onClick={() => {
                        setCsvImportSetId(set.id);
                        setCsvImportTargetCategoryId('__new__');
                        setCsvImportNewCategoryName('');
                        setCsvImportRows([]);
                        setCsvImportError('');
                        setScreen('import-csv');
                      }}
                      className="w-full mt-4 py-3 rounded-xl bg-emerald-500/10 text-emerald-400 border border-emerald-500/40 font-bold hover:bg-emerald-500/20 transition-all"
                    >
                      📄 Import dari File CSV
                    </button>
                  </div>

                  <div className="space-y-4">
                    {set.categories.length === 0 ? (
                      <div className="glass-card rounded-2xl p-12 text-center"><p className="text-slate-400">Belum ada kategori. Buat yang pertama!</p></div>
                    ) : (
                      set.categories.map(category => (
                        <div key={category.id} className="glass-card rounded-2xl p-6 hover:shadow-xl transition-all group" style={{ borderColor: `${category.color.bg}40` }}>
                          <div className="p-3 rounded-lg mb-4" style={{ backgroundColor: `${category.color.bg}20` }}>
                            <h4 className="font-bold text-lg" style={{ color: category.color.bg }}>{category.name}</h4>
                            <p className="text-sm text-slate-400 mt-1">{category.vocabulary.length} kosakata</p>
                          </div>
                          <div className="flex gap-3">
                            <button
                              onClick={() => { setCurrentCategoryId(category.id); setScreen('edit-category'); }}
                              className="flex-1 py-2 px-4 text-white rounded-lg font-semibold transition-colors"
                              style={{ backgroundColor: `${category.color.bg}30`, color: category.color.bg, border: `1px solid ${category.color.bg}50` }}
                            >
                              Edit Kosakata
                            </button>
                            <button
                              onClick={() => handleDeleteCategory(set.id, category.id)}
                              className="px-4 py-2 rounded-lg font-semibold bg-rose-500/10 text-rose-400 border border-rose-500/30 hover:bg-rose-500/20 transition-colors flex items-center gap-2"
                              title="Hapus kategori ini beserta semua kosakatanya"
                            >
                              <Icon.Trash2 className="w-4 h-4" />
                              Hapus Kategori
                            </button>
                          </div>
                        </div>
                      ))
                    )}
                  </div>
                </div>
              </div>
            );
          }

          if (screen === 'import-csv') {
            const set = vocabSets.find(s => s.id === csvImportSetId);
            if (!set) return null;
            const duplicateCount = csvImportRows.filter(r => r.isDuplicate).length;
            const includedCount = csvImportRows.filter(r => r.include).length;

            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6">
                <div className="max-w-4xl mx-auto">
                  <div className="flex justify-between items-center mb-8">
                    <div>
                      <p className="text-sm text-slate-400">{set.name}</p>
                      <h2 className="text-4xl font-bold text-white mt-2">Import dari CSV/Excel</h2>
                    </div>
                    <button
                      onClick={() => { setCsvImportRows([]); setCsvImportError(''); setScreen('manage-categories'); }}
                      className="px-6 py-3 bg-slate-700 text-white rounded-lg font-semibold hover:bg-slate-600 transition-colors"
                    >
                      ← Batal
                    </button>
                  </div>

                  <div className="glass-card rounded-2xl p-8 mb-6">
                    <h3 className="text-xl font-bold text-white mb-4">1. Masukkan ke kategori mana?</h3>
                    <select
                      value={csvImportTargetCategoryId}
                      onChange={(e) => { setCsvImportTargetCategoryId(e.target.value); setCsvImportRows([]); setCsvImportError(''); }}
                      className="w-full px-6 py-4 rounded-xl bg-slate-800/50 border border-slate-600 text-white focus:border-emerald-500 focus:outline-none transition-colors mb-4"
                    >
                      <option value="__new__">+ Buat kategori baru</option>
                      {set.categories.map(c => (
                        <option key={c.id} value={c.id}>{c.name} ({c.vocabulary.length} kosakata)</option>
                      ))}
                    </select>
                    {csvImportTargetCategoryId === '__new__' && (
                      <input
                        type="text"
                        value={csvImportNewCategoryName}
                        onChange={(e) => setCsvImportNewCategoryName(e.target.value)}
                        placeholder="Nama kategori baru"
                        className="w-full px-6 py-4 rounded-xl bg-slate-800/50 border border-slate-600 text-white placeholder-slate-500 focus:border-emerald-500 focus:outline-none transition-colors"
                      />
                    )}
                  </div>

                  <div className="glass-card rounded-2xl p-8 mb-6">
                    <h3 className="text-xl font-bold text-white mb-2">2. Upload file Excel atau CSV</h3>
                    <p className="text-sm text-slate-400 mb-4">
                      Format: 2 kolom, <span className="text-slate-300 font-medium">kata</span> lalu <span className="text-slate-300 font-medium">definisi</span>. File Excel (.xlsx/.xls) langsung didukung — tidak perlu diubah ke CSV dulu. Kalau file punya beberapa sheet, hanya sheet pertama yang dibaca.
                    </p>
                    <input
                      type="file"
                      accept=".csv,.xlsx,.xls,text/csv,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet,application/vnd.ms-excel"
                      ref={csvFileInputRef}
                      onChange={handleSpreadsheetFileSelected}
                      className="hidden"
                    />
                    <button
                      onClick={() => csvFileInputRef.current?.click()}
                      disabled={csvImportTargetCategoryId === '__new__' && !csvImportNewCategoryName.trim()}
                      className="w-full py-3 rounded-xl bg-emerald-500/10 text-emerald-400 border border-emerald-500/40 font-bold hover:bg-emerald-500/20 transition-all disabled:opacity-40 disabled:cursor-not-allowed"
                    >
                      📄 Pilih File Excel/CSV
                    </button>
                    {csvImportTargetCategoryId === '__new__' && !csvImportNewCategoryName.trim() && (
                      <p className="text-xs text-amber-400 mt-2">Isi nama kategori baru dulu di atas sebelum upload file.</p>
                    )}
                    {csvImportError && (
                      <div className="mt-4 p-4 rounded-xl bg-rose-500/10 border border-rose-500/30 text-rose-300 text-sm">{csvImportError}</div>
                    )}
                  </div>

                  {csvImportRows.length > 0 && (
                    <div className="glass-card rounded-2xl p-8">
                      <div className="flex justify-between items-center mb-4">
                        <h3 className="text-xl font-bold text-white">3. Preview ({csvImportRows.length} baris ditemukan)</h3>
                        <span className="text-sm text-slate-400">{includedCount} akan diimpor</span>
                      </div>
                      {duplicateCount > 0 && (
                        <div className="mb-4 p-4 rounded-xl bg-amber-500/10 border border-amber-500/30 text-amber-300 text-sm">
                          ⚠️ {duplicateCount} kata terdeteksi sudah ada (di kategori tujuan atau berulang di file). Sudah otomatis di-uncheck — centang manual kalau tetap ingin ditambahkan sebagai duplikat.
                        </div>
                      )}
                      <div className="space-y-2 max-h-96 overflow-y-auto pr-2 mb-6">
                        {csvImportRows.map((row, idx) => (
                          <div key={idx} className={`flex items-start gap-3 p-3 rounded-lg ${row.isDuplicate ? 'bg-amber-500/5 border border-amber-500/20' : 'bg-slate-800/30'}`}>
                            <input type="checkbox" checked={row.include} onChange={() => toggleCsvRowInclude(idx)} className="mt-1 w-4 h-4 accent-emerald-500 cursor-pointer" />
                            <div className="flex-1 min-w-0">
                              <div className="flex items-center gap-2">
                                <p className="font-semibold text-white truncate">{row.word}</p>
                                {row.isDuplicate && <span className="text-xs px-2 py-0.5 rounded bg-amber-500/20 text-amber-300 flex-shrink-0">duplikat</span>}
                              </div>
                              <p className="text-sm text-slate-400 truncate">{row.definition}</p>
                            </div>
                          </div>
                        ))}
                      </div>
                      <button
                        onClick={handleConfirmCsvImport}
                        disabled={includedCount === 0}
                        className="w-full py-3 rounded-xl bg-gradient-to-r from-emerald-500 to-teal-500 text-white font-bold hover:shadow-lg hover:shadow-emerald-500/50 transition-all disabled:opacity-40 disabled:cursor-not-allowed"
                      >
                        Import {includedCount} Kosakata
                      </button>
                    </div>
                  )}
                </div>
              </div>
            );
          }

          if (screen === 'edit-category') {
            const set = vocabSets.find(s => s.id === currentSetId);
            const category = set?.categories.find(c => c.id === currentCategoryId);
            if (!set || !category) return null;
            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6">
                <div className="max-w-4xl mx-auto">
                  <div className="flex justify-between items-center mb-8">
                    <div>
                      <p className="text-sm text-slate-400">{set.name}</p>
                      <h2 className="text-4xl font-bold text-white mt-2">{category.name}</h2>
                    </div>
                    <button onClick={() => setScreen('manage-categories')} className="px-6 py-3 bg-slate-700 text-white rounded-lg font-semibold hover:bg-slate-600 transition-colors">← Kembali</button>
                  </div>

                  <div className="glass-card rounded-2xl p-8 mb-8">
                    <h3 className="text-2xl font-bold text-white mb-6">Tambah Kosakata</h3>
                    <input
                      type="text"
                      value={newWord}
                      onChange={(e) => setNewWord(e.target.value)}
                      placeholder="Kata"
                      maxLength="100"
                      className="w-full px-6 py-4 rounded-xl bg-slate-800/50 border border-slate-600 text-white placeholder-slate-500 focus:border-teal-500 focus:outline-none mb-4 transition-colors"
                    />
                    <textarea
                      value={newDefinition}
                      onChange={(e) => setNewDefinition(e.target.value)}
                      placeholder="Definisi"
                      maxLength="500"
                      rows="3"
                      className="w-full px-6 py-4 rounded-xl bg-slate-800/50 border border-slate-600 text-white placeholder-slate-500 focus:border-teal-500 focus:outline-none mb-4 transition-colors resize-none"
                    />
                    <div className="flex justify-between text-xs text-slate-400 mb-4">
                      <span>{newWord.length}/100</span>
                      <span>{newDefinition.length}/500</span>
                    </div>
                    <div className="flex gap-3">
                      <button
                        onClick={handleClearVocabForm}
                        className="px-6 py-3 rounded-xl bg-slate-700 text-white font-bold hover:bg-slate-600 transition-colors flex items-center gap-2"
                        title="Kosongkan form (kata & definisi yang sedang diketik)"
                      >
                        <Icon.Trash2 className="w-5 h-5" /> Hapus
                      </button>
                      <button
                        onClick={() => handleAddVocab(set.id, category.id)}
                        className="flex-1 py-3 rounded-xl bg-gradient-to-r from-teal-500 to-cyan-500 text-white font-bold hover:shadow-lg hover:shadow-teal-500/50 transition-all"
                      >
                        Simpan Kosakata
                      </button>
                    </div>
                    <p className="text-sm text-slate-400 mt-6">Total: {category.vocabulary.length}/1500</p>
                  </div>

                  <div className="glass-card rounded-2xl p-8">
                    <h3 className="text-2xl font-bold text-white mb-6">Daftar Kosakata ({category.vocabulary.length})</h3>
                    {category.vocabulary.length === 0 ? (
                      <p className="text-center text-slate-400 py-8">Belum ada kosakata. Tambahkan sekarang!</p>
                    ) : (
                      <div className="space-y-3 max-h-96 overflow-y-auto pr-4">
                        {category.vocabulary.map((vocab, idx) => (
                          <div key={vocab.id} className="bg-slate-800/30 rounded-lg p-4 hover:bg-slate-800/50 transition-colors group flex justify-between items-start">
                            <div className="flex-1">
                              <div className="flex items-center gap-3 mb-2">
                                <span className="text-xs text-slate-500 bg-slate-700 px-2 py-1 rounded">#{idx + 1}</span>
                                <p className="font-semibold text-white">{vocab.word}</p>
                              </div>
                              <p className="text-sm text-slate-400">{vocab.definition}</p>
                            </div>
                            <button
                              onClick={() => handleDeleteVocab(set.id, category.id, vocab.id)}
                              className="p-2 hover:bg-rose-500/20 rounded-lg text-rose-400 transition-colors opacity-0 group-hover:opacity-100"
                            >
                              <Icon.Trash2 className="w-4 h-4" />
                            </button>
                          </div>
                        ))}
                      </div>
                    )}
                  </div>
                </div>
              </div>
            );
          }

          if (screen === 'select-category') {
            const set = vocabSets.find(s => s.id === currentSetId);
            if (!set) return null;
            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6">
                <div className="max-w-2xl mx-auto">
                  <div className="flex justify-between items-center mb-8">
                    <h2 className="text-4xl font-bold text-white">Pilih Kategori</h2>
                    <button onClick={() => setScreen('home')} className="px-6 py-3 bg-slate-700 text-white rounded-lg font-semibold hover:bg-slate-600 transition-colors">← Kembali</button>
                  </div>
                  <div className="space-y-4">
                    {set.categories.map(category => (
                      <button
                        key={category.id}
                        onClick={() => { setCurrentCategoryId(category.id); setScreen('games'); }}
                        className="glass-card rounded-2xl p-6 w-full text-left hover:shadow-xl hover:border-teal-500/30 transition-all group border border-slate-700"
                        style={{ borderColor: `${category.color.bg}40` }}
                      >
                        <div className="flex items-center justify-between">
                          <div>
                            <div className="inline-block px-4 py-2 rounded-lg font-bold mb-3" style={{ backgroundColor: `${category.color.bg}20`, color: category.color.bg }}>{category.name}</div>
                            <p className="text-sm text-slate-400">{category.vocabulary.length} kosakata</p>
                          </div>
                          <Icon.ChevronRight className="w-6 h-6 text-slate-400 group-hover:text-teal-400 transition-colors" />
                        </div>
                      </button>
                    ))}
                  </div>
                </div>
              </div>
            );
          }

          if (screen === 'games') {
            const set = vocabSets.find(s => s.id === currentSetId);
            const category = set?.categories.find(c => c.id === currentCategoryId);
            if (!set || !category || category.vocabulary.length === 0) return null;

            const games = [
              { id: 'flashcard', name: 'Kartu Flash', description: 'Flip kartu untuk lihat definisi', icon: <Icon.Brain className="w-8 h-8" /> },
              { id: 'quiz', name: 'Kuis Kosakata', description: 'Pilih jawaban yang benar', icon: <Icon.Target className="w-8 h-8" /> },
              { id: 'match', name: 'Catur Jawa', description: 'Pilih kata dari grid', icon: <Icon.Grid className="w-8 h-8" /> },
              { id: 'memory', name: 'Game Memori', description: 'Cocokkan kata dan definisi', icon: <Icon.Brain className="w-8 h-8" /> }
            ];

            return (
              <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6">
                <div className="max-w-4xl mx-auto">
                  <div className="flex justify-between items-center mb-8">
                    <div>
                      <p className="text-sm text-slate-400">{set.name}</p>
                      <h2 className="text-4xl font-bold text-white mt-2">{category.name}</h2>
                      <p className="text-sm text-slate-400 mt-1">{category.vocabulary.length} kosakata</p>
                    </div>
                    <button onClick={() => setScreen('select-category')} className="px-6 py-3 bg-slate-700 text-white rounded-lg font-semibold hover:bg-slate-600 transition-colors">← Kembali</button>
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    {games.map(game => (
                      <button
                        key={game.id}
                        onClick={() => handleStartGame(game.id)}
                        className="glass-card rounded-2xl p-8 text-left hover:shadow-2xl hover:shadow-teal-500/10 transition-all group border border-slate-700 hover:border-teal-500/30"
                      >
                        <div className="text-white group-hover:text-teal-300 transition-colors mb-4">{game.icon}</div>
                        <h3 className="text-2xl font-bold text-white mb-2">{game.name}</h3>
                        <p className="text-sm text-slate-300 mb-4">{game.description}</p>
                        <div className="flex items-center gap-2 text-sm font-semibold text-teal-400 group-hover:text-teal-300 transition-colors">
                          Mulai <Icon.ChevronRight className="w-4 h-4" />
                        </div>
                      </button>
                    ))}
                  </div>
                </div>
              </div>
            );
          }

          if (screen === 'practice') {
            if (!gameState) return null;

            if (gameState.finished) {
              const percentage = Math.round((gameState.finalScore / gameState.total) * 100);
              return (
                <div className="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-900 p-6 flex flex-col items-center justify-center">
                  <div className="w-full max-w-2xl">
                    <div className="glass-card rounded-3xl p-12 text-center">
                      <p className="text-6xl mb-6">🎉</p>
                      <h2 className="text-4xl font-bold text-white mb-2">Selesai!</h2>
                      <p className="text-slate-400 mb-8">Kamu menyelesaikan challenge</p>
                      <div className="mb-8">
                        <p className="text-6xl font-black text-transparent bg-clip-text bg-gradient-to-r from-emerald-400 to-teal-400">{gameState.finalScore}/{gameState.total}</p>
                        <p className="text-2xl font-semibold text-slate-300 mt-4">{percentage}%</p>
                      </div>
                      <button
                        onClick={() => { setGameState(null); setCurrentGameType(null); setScreen('games'); }}
                        className="w-full py-3 rounded-xl bg-gradient-to-r from-teal-500 to-cyan-500 text-white font-bold hover:shadow-lg hover:shadow-teal-500/50 transition-all"
                      >
                        Main Lagi
                      </button>
                    </div>
                  </div>
                </div>
              );
            }

            if (currentGameType === 'flashcard') return <FlashcardGame />;
            if (currentGameType === 'quiz') return <QuizGame />;
            if (currentGameType === 'match') return <MatchGame />;
            if (currentGameType === 'memory') return <MemoryGame />;
          }

          return null;
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<VocaLearn />);
    </script>
</body>
</html>
