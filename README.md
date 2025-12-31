import React, { useState, useEffect, useMemo } from 'react';
import { Search, Settings, ArrowLeft, Book, Moon, Sun, Type, Globe, Music, X } from 'lucide-react';

// --- ДАНІ (Тут ви будете додавати свої пісні) ---
const INITIAL_SONGS = [
  {
    id: 1,
    number: 1,
    book: "Збірник Відродження",
    versions: {
      ua: {
        title: "Благодать",
        text: `О благодать, спасенний я,\nЗ безодні я прозрів.\nБув мертвий - чую голос я,\nСліпий був - і прозрів.\n\nВ часи тривог і боротьби\nТи помагав мені.\nІ знаю я - в моїй журбі\nТи був при мені.`
      },
      en: {
        title: "Amazing Grace",
        text: `Amazing grace! How sweet the sound\nThat saved a wretch like me!\nI once was lost, but now am found;\nWas blind, but now I see.`
      }
    }
  },
  {
    id: 2,
    number: 2,
    book: "Пісні Хвали",
    versions: {
      ua: {
        title: "Тиша навкруги",
        text: `Тиша навкруги, сплять в росі луги,\nТільки ти не спиш, Господа молиш.\nНа колінах ти, в сльозах скорботи,\nЗа мій гріх тяжкий, там в саду один.\n\nПриспів:\nО, Ісусе мій, Ти - Спаситель мій,\nТи за мене страждав, Ти життя віддав,\nЩоб я вічно жив, Господа хвалив,\nО, Ісусе мій, Ти - Спаситель мій.`
      },
      en: {
        title: "Silence all around",
        text: "(English text placeholder...)\nSilence all around, meadows sleep in dew..."
      }
    }
  },
  {
    id: 3,
    number: 15,
    book: "Збірник Відродження",
    versions: {
      ua: {
        title: "Великий Бог",
        text: `Коли я бачу, як на небі синім\nЗорі горять, де Ти шляхи проклав,\nЯк сонце гріє, як у час осінній\nЛист опада, - я славлю Твої діла.\n\nПриспів:\nТоді співає мій дух Тобі:\nВеликий Бог, Великий Бог!\nТоді співає мій дух Тобі:\nВеликий Бог, Великий Бог!`
      }
    }
  },
  {
    id: 4,
    number: 4,
    book: "Молодіжний",
    versions: {
      ua: {
        title: "Лиш Тобі",
        text: `Лиш Тобі, лиш Тобі,\nСлава, Господи, Тобі!\nЗа чудовий Твій світ,\nЗа дарунок життя,\nЛиш Тобі, лиш Тобі слава вся!`
      }
    }
  }
];

// --- ПЕРЕКЛАДИ ІНТЕРФЕЙСУ ---
const UI_STRINGS = {
  ua: {
    appTitle: "Співанник",
    settings: "Налаштування",
    searchPlaceholder: "Пошук (назва або текст)...",
    appearance: "Вигляд",
    theme: "Тема",
    light: "Світла",
    dark: "Темна",
    content: "Зміст",
    songBook: "Книга пісень",
    allBooks: "Всі книги",
    sortBy: "Сортувати за",
    sortNumber: "Номером",
    sortTitle: "Назвою",
    language: "Мова",
    appLang: "Мова застосунку",
    songLang: "Мова пісень",
    fontSize: "Розмір тексту",
    noResults: "Пісень не знайдено",
    back: "Назад"
  },
  en: {
    appTitle: "Songbook",
    settings: "Settings",
    searchPlaceholder: "Search (title or lyrics)...",
    appearance: "Appearance",
    theme: "Theme",
    light: "Light",
    dark: "Dark",
    content: "Content",
    songBook: "Song Book",
    allBooks: "All Books",
    sortBy: "Sort by",
    sortNumber: "Number",
    sortTitle: "Title",
    language: "Language",
    appLang: "App Language",
    songLang: "Song Language",
    fontSize: "Font Size",
    noResults: "No songs found",
    back: "Back"
  }
};

const App = () => {
  // --- СТАН (STATE) ---
  const [view, setView] = useState('list'); // 'list', 'song', 'settings'
  const [activeSongId, setActiveSongId] = useState(null);
  
  // Налаштування
  const [settings, setSettings] = useState({
    appLang: 'ua',      // 'ua' | 'en'
    songLang: 'ua',     // 'ua' | 'en'
    theme: 'light',     // 'light' | 'dark'
    selectedBook: 'all',
    sortBy: 'number',   // 'number' | 'title'
    fontSize: 18        // px
  });

  const [searchQuery, setSearchQuery] = useState('');

  // --- ЕФЕКТИ ---
  
  // Зміна теми (Dark Mode)
  useEffect(() => {
    if (settings.theme === 'dark') {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
  }, [settings.theme]);

  // Отримання унікальних книг для фільтру
  const availableBooks = useMemo(() => {
    const books = new Set(INITIAL_SONGS.map(s => s.book));
    return Array.from(books);
  }, []);

  // Тексти інтерфейсу
  const t = UI_STRINGS[settings.appLang];

  // --- ЛОГІКА ФІЛЬТРАЦІЇ ТА СОРТУВАННЯ ---
  const filteredSongs = useMemo(() => {
    let result = INITIAL_SONGS;

    // 1. Фільтр по книзі
    if (settings.selectedBook !== 'all') {
      result = result.filter(s => s.book === settings.selectedBook);
    }

    // 2. Пошук (Назва або Текст)
    if (searchQuery.trim()) {
      const query = searchQuery.toLowerCase();
      result = result.filter(song => {
        // Отримуємо дані пісні по вибраній мові, або фолбек на першу доступну
        const version = song.versions[settings.songLang] || Object.values(song.versions)[0];
        if (!version) return false;
        
        const titleMatch = version.title.toLowerCase().includes(query);
        const textMatch = version.text.toLowerCase().includes(query);
        const numberMatch = song.number.toString().includes(query);

        return titleMatch || textMatch || numberMatch;
      });
    }

    // 3. Сортування
    result = [...result].sort((a, b) => {
      if (settings.sortBy === 'number') {
        return a.number - b.number;
      } else {
        const titleA = (a.versions[settings.songLang]?.title || "").toLowerCase();
        const titleB = (b.versions[settings.songLang]?.title || "").toLowerCase();
        return titleA.localeCompare(titleB);
      }
    });

    return result;
  }, [settings.selectedBook, settings.songLang, settings.sortBy, searchQuery]);

  // --- ОБРОБНИКИ ПОДІЙ ---
  const handleSongClick = (id) => {
    setActiveSongId(id);
    setView('song');
  };

  const goBack = () => {
    setView('list');
    setActiveSongId(null);
  };

  const toggleSettings = () => {
    if (view === 'settings') setView('list');
    else setView('settings');
  };

  // Отримання поточної пісні для відображення
  const activeSong = useMemo(() => {
    if (!activeSongId) return null;
    return INITIAL_SONGS.find(s => s.id === activeSongId);
  }, [activeSongId]);

  // --- КОМПОНЕНТИ ---

  // Компонент перемикача (Toggle/Select)
  const SettingRow = ({ icon: Icon, label, children }) => (
    <div className="flex flex-col py-3 border-b border-gray-200 dark:border-gray-700 last:border-0">
      <div className="flex items-center mb-2 text-gray-700 dark:text-gray-300">
        <Icon size={18} className="mr-2" />
        <span className="font-medium">{label}</span>
      </div>
      <div className="pl-6">
        {children}
      </div>
    </div>
  );

  // --- RENDER: SETTINGS ---
  if (view === 'settings') {
    return (
      <div className="min-h-screen bg-gray-50 dark:bg-slate-900 text-gray-900 dark:text-gray-100 transition-colors duration-200">
        {/* Header */}
        <div className="sticky top-0 bg-white dark:bg-slate-800 shadow-sm z-10 px-4 py-3 flex items-center justify-between">
          <button onClick={() => setView('list')} className="p-2 -ml-2 rounded-full hover:bg-gray-100 dark:hover:bg-slate-700">
            <ArrowLeft size={24} />
          </button>
          <h1 className="text-lg font-bold">{t.settings}</h1>
          <div className="w-8"></div> {/* Spacer */}
        </div>

        <div className="p-4 max-w-md mx-auto">
          <div className="bg-white dark:bg-slate-800 rounded-xl shadow-sm p-4">
            
            {/* Theme */}
            <SettingRow icon={settings.theme === 'light' ? Sun : Moon} label={t.appearance}>
              <div className="flex bg-gray-100 dark:bg-slate-700 rounded-lg p-1">
                <button 
                  onClick={() => setSettings({...settings, theme: 'light'})}
                  className={`flex-1 py-1.5 text-sm rounded-md transition-all ${settings.theme === 'light' ? 'bg-white shadow text-blue-600' : 'text-gray-500 dark:text-gray-400'}`}
                >
                  {t.light}
                </button>
                <button 
                  onClick={() => setSettings({...settings, theme: 'dark'})}
                  className={`flex-1 py-1.5 text-sm rounded-md transition-all ${settings.theme === 'dark' ? 'bg-slate-600 shadow text-white' : 'text-gray-500 dark:text-gray-400'}`}
                >
                  {t.dark}
                </button>
              </div>
            </SettingRow>

            {/* Font Size */}
            <SettingRow icon={Type} label={`${t.fontSize}: ${settings.fontSize}px`}>
              <input 
                type="range" 
                min="12" 
                max="32" 
                step="2"
                value={settings.fontSize}
                onChange={(e) => setSettings({...settings, fontSize: Number(e.target.value)})}
                className="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer dark:bg-gray-700 accent-blue-600"
              />
            </SettingRow>

            {/* Language Settings */}
            <SettingRow icon={Globe} label={t.language}>
              <div className="space-y-3">
                <div className="flex justify-between items-center">
                  <span className="text-sm text-gray-500 dark:text-gray-400">{t.appLang}</span>
                  <select 
                    value={settings.appLang}
                    onChange={(e) => setSettings({...settings, appLang: e.target.value})}
                    className="bg-gray-100 dark:bg-slate-700 border-none rounded-md px-3 py-1 text-sm focus:ring-2 focus:ring-blue-500"
                  >
                    <option value="ua">Українська</option>
                    <option value="en">English</option>
                  </select>
                </div>
                <div className="flex justify-between items-center">
                  <span className="text-sm text-gray-500 dark:text-gray-400">{t.songLang}</span>
                  <select 
                    value={settings.songLang}
                    onChange={(e) => setSettings({...settings, songLang: e.target.value})}
                    className="bg-gray-100 dark:bg-slate-700 border-none rounded-md px-3 py-1 text-sm focus:ring-2 focus:ring-blue-500"
                  >
                    <option value="ua">Українська</option>
                    <option value="en">English</option>
                  </select>
                </div>
              </div>
            </SettingRow>

            {/* Content Filters */}
            <SettingRow icon={Book} label={t.content}>
               <div className="space-y-3">
                 <div className="flex flex-col gap-1">
                   <span className="text-sm text-gray-500 dark:text-gray-400">{t.songBook}</span>
                   <select 
                      value={settings.selectedBook}
                      onChange={(e) => setSettings({...settings, selectedBook: e.target.value})}
                      className="w-full bg-gray-100 dark:bg-slate-700 border-none rounded-md px-3 py-2 text-sm focus:ring-2 focus:ring-blue-500"
                    >
                      <option value="all">{t.allBooks}</option>
                      {availableBooks.map(book => (
                        <option key={book} value={book}>{book}</option>
                      ))}
                    </select>
                 </div>
                 
                 <div className="flex justify-between items-center pt-2">
                    <span className="text-sm text-gray-500 dark:text-gray-400">{t.sortBy}</span>
                    <div className="flex bg-gray-100 dark:bg-slate-700 rounded-lg p-0.5">
                      <button 
                        onClick={() => setSettings({...settings, sortBy: 'number'})}
                        className={`px-3 py-1 text-xs rounded transition-all ${settings.sortBy === 'number' ? 'bg-white dark:bg-slate-600 shadow' : ''}`}
                      >
                        #
                      </button>
                      <button 
                        onClick={() => setSettings({...settings, sortBy: 'title'})}
                        className={`px-3 py-1 text-xs rounded transition-all ${settings.sortBy === 'title' ? 'bg-white dark:bg-slate-600 shadow' : ''}`}
                      >
                        ABC
                      </button>
                    </div>
                 </div>
               </div>
            </SettingRow>

          </div>
        </div>
      </div>
    );
  }

  // --- RENDER: SONG VIEW ---
  if (view === 'song' && activeSong) {
    // Get the version based on language setting, or fallback to first available
    const songData = activeSong.versions[settings.songLang] || Object.values(activeSong.versions)[0];
    const isFallback = !activeSong.versions[settings.songLang];

    return (
      <div className="min-h-screen flex flex-col bg-white dark:bg-slate-900 text-gray-900 dark:text-gray-100 transition-colors duration-200">
        {/* Navbar */}
        <div className="sticky top-0 bg-white/95 dark:bg-slate-900/95 backdrop-blur-sm shadow-sm z-10 px-4 py-3 flex items-center justify-between">
          <button onClick={goBack} className="p-2 -ml-2 rounded-full hover:bg-gray-100 dark:hover:bg-slate-800">
            <ArrowLeft size={24} />
          </button>
          <div className="text-center">
            <h1 className="text-lg font-bold leading-none">#{activeSong.number}</h1>
            <span className="text-xs text-gray-500 dark:text-gray-400">{activeSong.book}</span>
          </div>
          <button onClick={() => setSettings(s => ({...s, fontSize: s.fontSize < 30 ? s.fontSize + 2 : 14}))} className="p-2 -mr-2 text-gray-500">
            <Type size={20} />
          </button>
        </div>

        {/* Content */}
        <div className="flex-1 p-5 max-w-2xl mx-auto w-full">
          {isFallback && (
            <div className="mb-4 p-2 bg-yellow-50 dark:bg-yellow-900/30 text-yellow-800 dark:text-yellow-200 text-xs rounded text-center">
              Переклад відсутній, показано оригінал
            </div>
          )}
          
          <h2 className="text-2xl font-bold mb-6 text-center text-blue-600 dark:text-blue-400">
            {songData.title}
          </h2>
          
          <div 
            className="whitespace-pre-wrap leading-relaxed pb-12"
            style={{ fontSize: `${settings.fontSize}px` }}
          >
            {songData.text}
          </div>
        </div>
      </div>
    );
  }

  // --- RENDER: LIST VIEW (MAIN) ---
  return (
    <div className="min-h-screen flex flex-col bg-gray-50 dark:bg-slate-900 text-gray-900 dark:text-gray-100 transition-colors duration-200">
      
      {/* Header */}
      <div className="sticky top-0 bg-blue-600 dark:bg-slate-800 shadow-md z-20">
        <div className="px-4 py-3 flex justify-between items-center text-white">
          <div className="flex items-center gap-2">
            <Music size={24} />
            <h1 className="text-xl font-bold">{t.appTitle}</h1>
          </div>
          <button onClick={toggleSettings} className="p-2 rounded-full hover:bg-white/10 transition-colors">
            <Settings size={22} />
          </button>
        </div>

        {/* Search Bar */}
        <div className="px-4 pb-3">
          <div className="relative">
            <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
              <Search size={18} className="text-gray-400" />
            </div>
            <input
              type="text"
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              placeholder={t.searchPlaceholder}
              className="w-full pl-10 pr-10 py-2.5 rounded-lg bg-white dark:bg-slate-700 text-gray-900 dark:text-white placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-300 dark:focus:ring-blue-500 shadow-sm"
            />
            {searchQuery && (
              <button 
                onClick={() => setSearchQuery('')}
                className="absolute inset-y-0 right-0 pr-3 flex items-center text-gray-400 hover:text-gray-600"
              >
                <X size={16} />
              </button>
            )}
          </div>
        </div>
      </div>

      {/* List */}
      <div className="flex-1 p-2 max-w-2xl mx-auto w-full">
        {filteredSongs.length > 0 ? (
          <div className="space-y-2">
            {filteredSongs.map((song) => {
               const songData = song.versions[settings.songLang] || Object.values(song.versions)[0];
               return (
                <div 
                  key={song.id}
                  onClick={() => handleSongClick(song.id)}
                  className="bg-white dark:bg-slate-800 p-4 rounded-xl shadow-sm hover:shadow-md active:scale-[0.99] transition-all cursor-pointer flex items-center gap-4"
                >
                  <div className="flex-shrink-0 w-10 h-10 bg-blue-100 dark:bg-blue-900/30 text-blue-600 dark:text-blue-400 rounded-full flex items-center justify-center font-bold text-lg">
                    {song.number}
                  </div>
                  <div className="flex-1 min-w-0">
                    <h3 className="font-semibold text-lg truncate pr-2">
                      {songData.title}
                    </h3>
                    <p className="text-xs text-gray-500 dark:text-gray-400 truncate">
                      {song.book} • {songData.text.substring(0, 30).replace(/\n/g, ' ')}...
                    </p>
                  </div>
                </div>
               );
            })}
          </div>
        ) : (
          <div className="flex flex-col items-center justify-center pt-20 text-gray-400">
            <Search size={48} className="mb-4 opacity-50" />
            <p>{t.noResults}</p>
          </div>
        )}
      </div>
      
    </div>
  );
};

export default App;
