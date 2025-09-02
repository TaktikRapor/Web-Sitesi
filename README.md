import React, { useEffect, useMemo, useState } from "react";

// Basit haber sitesi (Türkçe). Tek dosya, Tailwind ile şık tasarım.
// Özellikler:
// - Manşet (öne çıkan haber)
// - Kategori filtresi, arama, sıralama (Yeniden/En çok Okunan)
// - Haber kartları ve detay sayfası
// - Basit "Yönetici" modali ile haber ekleme (localStorage'a kaydeder)
// - Karanlık/aydınlık tema anahtarı

// Yardımcılar
const slugify = (s) =>
  s
    .toLowerCase()
    .replace(/ç/g, "c")
    .replace(/ğ/g, "g")
    .replace(/ı/g, "i")
    .replace(/ö/g, "o")
    .replace(/ş/g, "s")
    .replace(/ü/g, "u")
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)/g, "");

const nowISO = () => new Date().toISOString();

const defaultArticles = [
  {
    id: "1",
    title: "Yerel Yönetimlerden Yeni Ulaşım Hamlesi",
    summary:
      "Büyükşehir Belediyesi, toplu taşımada kapasiteyi artıracak yeni metro hattını açıkladı.",
    category: "Gündem",
    cover:
      "https://images.unsplash.com/photo-1544620347-c4fd4a3d5957?q=80&w=1600&auto=format&fit=crop",
    content:
      "Şehrin doğu ve batı yakasını birbirine bağlayacak hat için ilk kazma bu ay vurulacak. Projenin 2027'de tamamlanması bekleniyor.",
    createdAt: nowISO(),
    views: 128,
    author: "Haber Merkezi",
  },
  {
    id: "2",
    title: "Teknoloji Şirketlerinden Yapay Zekâ Atağı",
    summary:
      "Girişimler, üretken yapay zekâ araçlarını kamuya açarken regülasyon tartışmaları hızlandı.",
    category: "Teknoloji",
    cover:
      "https://images.unsplash.com/photo-1550751827-4bd374c3f58b?q=80&w=1600&auto=format&fit=crop",
    content:
      "Uzmanlara göre şirketler şeffaflık ve güvenlik konularına daha fazla yatırım yapacak.",
    createdAt: nowISO(),
    views: 342,
    author: "Tekno Masa",
  },
  {
    id: "3",
    title: "Spor Kulübü Avrupa'da Tarih Yazdı",
    summary:
      "Temsilcimiz, zorlu deplasmanda aldığı galibiyetle grupta liderliğe yükseldi.",
    category: "Spor",
    cover:
      "https://images.unsplash.com/photo-1517649763962-0c623066013b?q=80&w=1600&auto=format&fit=crop",
    content:
      "Teknik direktör maç sonrası, 'Disiplinli oyunumuz meyvesini verdi' dedi.",
    createdAt: nowISO(),
    views: 510,
    author: "Spor Servisi",
  },
];

const readLS = (key, fallback) => {
  try {
    const s = localStorage.getItem(key);
    return s ? JSON.parse(s) : fallback;
  } catch (e) {
    return fallback;
  }
};

const writeLS = (key, val) => {
  try {
    localStorage.setItem(key, JSON.stringify(val));
  } catch (e) {}
};

function useDarkMode() {
  const [dark, setDark] = useState(() => {
    if (typeof window === "undefined") return false;
    return readLS("news.dark", false);
  });
  useEffect(() => {
    const root = document.documentElement;
    if (dark) root.classList.add("dark");
    else root.classList.remove("dark");
    writeLS("news.dark", dark);
  }, [dark]);
  return [dark, setDark];
}

function ArticleForm({ onSave, onCancel }) {
  const [data, setData] = useState({
    title: "",
    summary: "",
    category: "Gündem",
    cover: "",
    content: "",
    author: "Editör",
  });

  const canSave = data.title.trim() && data.summary.trim() && data.content.trim();

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4">
      <div className="w-full max-w-2xl rounded-2xl bg-white p-6 shadow-2xl dark:bg-zinc-900">
        <div className="mb-4 flex items-center justify-between">
          <h3 className="text-xl font-semibold">Yeni Haber</h3>
          <button
            className="rounded-xl px-3 py-1 text-sm hover:bg-zinc-100 dark:hover:bg-zinc-800"
            onClick={onCancel}
          >
            Kapat
          </button>
        </div>
        <div className="grid gap-3">
          <input
            className="rounded-xl border p-2 dark:border-zinc-700 dark:bg-zinc-800"
            placeholder="Başlık"
            value={data.title}
            onChange={(e) => setData({ ...data, title: e.target.value })}
          />
          <input
            className="rounded-xl border p-2 dark:border-zinc-700 dark:bg-zinc-800"
            placeholder="Özet"
            value={data.summary}
            onChange={(e) => setData({ ...data, summary: e.target.value })}
          />
          <div className="grid grid-cols-2 gap-3">
            <select
              className="rounded-xl border p-2 dark:border-zinc-700 dark:bg-zinc-800"
              value={data.category}
              onChange={(e) => setData({ ...data, category: e.target.value })}
            >
              {[
                "Gündem",
                "Ekonomi",
                "Teknoloji",
                "Spor",
                "Kültür",
                "Dünya",
                "Yaşam",
              ].map((c) => (
                <option key={c}>{c}</option>
              ))}
            </select>
            <input
              className="rounded-xl border p-2 dark:border-zinc-700 dark:bg-zinc-800"
              placeholder="Kapak görseli URL (opsiyonel)"
              value={data.cover}
              onChange={(e) => setData({ ...data, cover: e.target.value })}
            />
          </div>
          <textarea
            className="min-h-[140px] rounded-xl border p-2 dark:border-zinc-700 dark:bg-zinc-800"
            placeholder="İçerik"
            value={data.content}
            onChange={(e) => setData({ ...data, content: e.target.value })}
          />
          <div className="grid grid-cols-2 gap-3">
            <input
              className="rounded-xl border p-2 dark:border-zinc-700 dark:bg-zinc-800"
              placeholder="Yazar"
              value={data.author}
              onChange={(e) => setData({ ...data, author: e.target.value })}
            />
            <div className="flex items-center justify-end gap-2">
              <button
                onClick={onCancel}
                className="rounded-xl border px-4 py-2 text-sm hover:bg-zinc-50 dark:border-zinc-700 dark:hover:bg-zinc-800"
              >
                İptal
              </button>
              <button
                disabled={!canSave}
                onClick={() =>
                  onSave({ ...data, id: crypto.randomUUID(), createdAt: nowISO(), views: 0 })
                }
                className={`rounded-xl px-4 py-2 text-sm text-white ${
                  canSave ? "bg-zinc-900 hover:bg-black dark:bg-zinc-100 dark:text-black" : "bg-zinc-300"
                }`}
              >
                Yayınla
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}

function ArticleCard({ a, onOpen }) {
  return (
    <article
      className="group overflow-hidden rounded-2xl border bg-white shadow-sm transition hover:shadow-md dark:border-zinc-800 dark:bg-zinc-900"
    >
      {a.cover && (
        <div className="relative h-48 w-full overflow-hidden">
          <img
            src={a.cover}
            alt={a.title}
            className="h-full w-full object-cover transition-transform duration-300 group-hover:scale-[1.03]"
            loading="lazy"
          />
        </div>
      )}
      <div className="flex flex-col gap-2 p-4">
        <div className="flex items-center gap-2 text-xs text-zinc-500 dark:text-zinc-400">
          <span className="rounded-full bg-zinc-100 px-2 py-0.5 dark:bg-zinc-800">{a.category}</span>
          <time>{new Date(a.createdAt).toLocaleDateString("tr-TR")}</time>
          <span>•</span>
          <span>{a.views} okuma</span>
        </div>
        <h3 className="line-clamp-2 text-lg font-semibold">{a.title}</h3>
        <p className="line-clamp-2 text-sm text-zinc-600 dark:text-zinc-300">{a.summary}</p>
        <button
          onClick={() => onOpen(a)}
          className="mt-1 self-start rounded-xl border px-3 py-1 text-sm hover:bg-zinc-50 dark:border-zinc-700 dark:hover:bg-zinc-800"
        >
          Oku
        </button>
      </div>
    </article>
  );
}

function ArticleView({ a, onBack }) {
  useEffect(() => {
    // Görüntülenme sayısını artır
    a.views = (a.views || 0) + 1;
  }, []);
  return (
    <div className="mx-auto max-w-3xl">
      <button
        onClick={onBack}
        className="mb-4 rounded-xl border px-3 py-1 text-sm hover:bg-zinc-50 dark:border-zinc-700 dark:hover:bg-zinc-800"
      >
        ← Geri
      </button>
      <h1 className="mb-2 text-3xl font-bold leading-tight">{a.title}</h1>
      <div className="mb-6 flex items-center gap-2 text-sm text-zinc-500 dark:text-zinc-400">
        <span className="rounded-full bg-zinc-100 px-2 py-0.5 dark:bg-zinc-800">{a.category}</span>
        <time>{new Date(a.createdAt).toLocaleString("tr-TR")}</time>
        <span>•</span>
        <span>{a.author}</span>
        <span>•</span>
        <span>{a.views} okuma</span>
      </div>
      {a.cover && (
        <img
          src={a.cover}
          alt={a.title}
          className="mb-6 h-80 w-full rounded-2xl object-cover"
        />
      )}
      <div className="prose max-w-none dark:prose-invert">
        <p>{a.content}</p>
      </div>
    </div>
  );
}

export default function NewsApp() {
  const [dark, setDark] = useDarkMode();
  const [articles, setArticles] = useState(() => readLS("news.articles", null) || defaultArticles);
  const [query, setQuery] = useState("");
  const [category, setCategory] = useState("Tümü");
  const [sort, setSort] = useState("Yeniler");
  const [featuredId, setFeaturedId] = useState(articles[0]?.id || null);
  const [showForm, setShowForm] = useState(false);
  const [view, setView] = useState({ type: "home", slug: null });

  useEffect(() => {
    writeLS("news.articles", articles);
  }, [articles]);

  const cats = useMemo(() => ["Tümü", ...Array.from(new Set(articles.map((a) => a.category)))], [articles]);

  const filtered = useMemo(() => {
    let list = [...articles];
    if (category !== "Tümü") list = list.filter((a) => a.category === category);
    if (query.trim()) {
      const q = query.toLowerCase();
      list = list.filter(
        (a) => a.title.toLowerCase().includes(q) || a.summary.toLowerCase().includes(q) || a.content.toLowerCase().includes(q)
      );
    }
    if (sort === "Yeniler") list.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
    else if (sort === "Popüler") list.sort((a, b) => (b.views || 0) - (a.views || 0));
    return list;
  }, [articles, category, query, sort]);

  const featured = articles.find((a) => a.id === featuredId) || filtered[0];

  const openArticle = (a) => {
    setView({ type: "article", slug: slugify(a.title) });
    // görüntülenme artışı için state güncelle
    setArticles((prev) => prev.map((x) => (x.id === a.id ? { ...x, views: (x.views || 0) + 1 } : x)));
  };

  const current = useMemo(() => {
    if (view.type === "article") {
      // slug ile bul
      return articles.find((a) => slugify(a.title) === view.slug) || null;
    }
    return null;
  }, [view, articles]);

  return (
    <div className="min-h-screen bg-zinc-50 text-zinc-900 antialiased dark:bg-zinc-950 dark:text-zinc-50">
      {/* Üstbar */}
      <header className="sticky top-0 z-40 border-b bg-white/80 backdrop-blur dark:border-zinc-800 dark:bg-zinc-950/80">
        <div className="mx-auto flex max-w-6xl items-center justify-between gap-4 p-4">
          <div className="flex items-center gap-3">
            <button
              onClick={() => setView({ type: "home", slug: null })}
              className="rounded-2xl bg-zinc-900 px-3 py-1.5 text-sm font-bold text-white dark:bg-zinc-100 dark:text-zinc-900"
            >
              HABERim
            </button>
            <nav className="hidden items-center gap-2 md:flex">
              {cats.map((c) => (
                <button
                  key={c}
                  onClick={() => setCategory(c)}
                  className={`rounded-xl px-3 py-1 text-sm ${
                    category === c
                      ? "bg-zinc-900 text-white dark:bg-zinc-100 dark:text-zinc-900"
                      : "hover:bg-zinc-100 dark:hover:bg-zinc-800"
                  }`}
                >
                  {c}
                </button>
              ))}
            </nav>
          </div>

          <div className="flex items-center gap-2">
            <div className="relative">
              <input
                className="w-44 rounded-xl border px-3 py-1.5 text-sm outline-none focus:ring-2 focus:ring-zinc-300 dark:border-zinc-700 dark:bg-zinc-900 md:w-64"
                placeholder="Ara…"
                value={query}
                onChange={(e) => setQuery(e.target.value)}
              />
            </div>
            <select
              className="rounded-xl border px-2 py-1.5 text-sm dark:border-zinc-700 dark:bg-zinc-900"
              value={sort}
              onChange={(e) => setSort(e.target.value)}
            >
              <option>Yeniler</option>
              <option>Popüler</option>
            </select>
            <button
              onClick={() => setDark((d) => !d)}
              className="rounded-xl border px-2 py-1.5 text-sm hover:bg-zinc-50 dark:border-zinc-700 dark:hover:bg-zinc-800"
              title="Tema değiştir"
            >
              {dark ? "🌙" : "☀️"}
            </button>
            <button
              onClick={() => setShowForm(true)}
              className="rounded-xl bg-emerald-600 px-3 py-1.5 text-sm font-medium text-white hover:bg-emerald-700"
            >
              + Haber Ekle
            </button>
          </div>
        </div>
      </header>

      {/* İçerik */}
      <main className="mx-auto max-w-6xl p-4">
        {view.type === "article" && current ? (
          <ArticleView a={current} onBack={() => setView({ type: "home", slug: null })} />
        ) : (
          <>
            {/* Manşet */}
            {featured && (
              <section className="mb-8 overflow-hidden rounded-3xl border bg-white shadow-sm dark:border-zinc-800 dark:bg-zinc-900">
                <div className="grid grid-cols-1 md:grid-cols-2">
                  <div className="order-2 p-6 md:order-1 md:p-8">
                    <div className="mb-2 flex items-center gap-2 text-xs text-zinc-500 dark:text-zinc-400">
                      <span className="rounded-full bg-zinc-100 px-2 py-0.5 dark:bg-zinc-800">{featured.category}</span>
                      <time>{new Date(featured.createdAt).toLocaleDateString("tr-TR")}</time>
                      <span>•</span>
                      <span>{featured.views} okuma</span>
                    </div>
                    <h2 className="mb-3 text-2xl font-bold md:text-3xl">{featured.title}</h2>
                    <p className="mb-4 text-zinc-600 dark:text-zinc-300">{featured.summary}</p>
                    <div className="flex gap-2">
                      <button
                        onClick={() => openArticle(featured)}
                        className="rounded-xl bg-zinc-900 px-4 py-2 text-sm font-medium text-white hover:bg-black dark:bg-zinc-100 dark:text-zinc-900"
                      >
                        Haberi Oku
                      </button>
                      <button
                        onClick={() => setFeaturedId(featured.id)}
                        className="rounded-xl border px-4 py-2 text-sm hover:bg-zinc-50 dark:border-zinc-700 dark:hover:bg-zinc-800"
                      >
                        Manşete Al
                      </button>
                    </div>
                  </div>
                  <div className="order-1 max-h-96 overflow-hidden md:order-2">
                    {featured.cover && (
                      <img src={featured.cover} alt={featured.title} className="h-full w-full object-cover" />
                    )}
                  </div>
                </div>
              </section>
            )}

            {/* Liste */}
            <section>
              <div className="mb-3 flex items-center justify-between">
                <h3 className="text-lg font-semibold">Son Haberler</h3>
                <div className="flex gap-2">
                  <select
                    className="rounded-xl border px-2 py-1.5 text-sm dark:border-zinc-700 dark:bg-zinc-900 md:hidden"
                    value={category}
                    onChange={(e) => setCategory(e.target.value)}
                  >
                    {cats.map((c) => (
                      <option key={c}>{c}</option>
                    ))}
                  </select>
                </div>
              </div>
              {filtered.length === 0 ? (
                <div className="rounded-2xl border p-8 text-center text-zinc-500 dark:border-zinc-800">
                  Aramanızla eşleşen haber bulunamadı.
                </div>
              ) : (
                <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
                  {filtered.map((a) => (
                    <ArticleCard key={a.id} a={a} onOpen={openArticle} />
                  ))}
                </div>
              )}
            </section>
          </>
        )}
      </main>

      {/* Altbilgi */}
      <footer className="mt-12 border-t p-6 text-center text-sm text-zinc-500 dark:border-zinc-800">
        © {new Date().getFullYear()} HABERim • Basit haber platformu — Yerel depolama ile çalışır.
      </footer>

      {showForm && (
        <ArticleForm
          onCancel={() => setShowForm(false)}
          onSave={(data) => {
            setArticles((prev) => [{ ...data }, ...prev]);
            setShowForm(false);
          }}
        />
      )}
    </div>
  );
}
