import React, { useState, useEffect } from 'react';
import { Search, ShoppingBag, Menu, X, Filter, ChevronLeft, Check, Plus } from 'lucide-react';

// ==========================================
// 1. MOCK DATABASE (Bisa ditambah secara dinamis)
// ==========================================
let GLOBAL_DB = [
  {
    _id: "p1",
    name: "Abaya Hitam Modern - Gentle Woman",
    brand: "NVP EXCLUSIVE",
    price: 350000,
    originalPrice: 450000,
    colors: ["Hitam Pink"],
    sizes: ["S", "M", "L", "XL"],
    image: "photo_6204060867754659801_w.jpg",
    description: "Abaya hitam modern yang elegan dengan aksen warna dusty pink di bagian lengan. Memberikan kesan kasual namun tetap anggun. Cocok dipadukan dengan tas Gentle Woman.",
    createdAt: "2024-03-01T00:00:00Z"
  },
  {
    _id: "p2",
    name: "Abaya Hitam Tali Lengan Pink",
    brand: "NVP EXCLUSIVE",
    price: 325000,
    originalPrice: 0,
    colors: ["Hitam Pink"],
    sizes: ["M", "L", "XL"],
    image: "photo_6204060867754659800_w.jpg",
    description: "Desain abaya kekinian dengan detail tali menyilang di bagian lengan berwarna pink. Sangat pas untuk gaya OOTD sehari-hari.",
    createdAt: "2024-03-02T00:00:00Z"
  },
  {
    _id: "p3",
    name: "Abaya Hitam Flowy A-Line",
    brand: "NVP EXCLUSIVE",
    price: 340000,
    originalPrice: 399000,
    colors: ["Hitam Pink"],
    sizes: ["All Size"],
    image: "photo_6204060867754659799_w.jpg",
    description: "Abaya berpotongan A-line yang jatuh dan sangat flowy saat digunakan. Dilengkapi kancing samping sebagai detail tambahan yang manis.",
    createdAt: "2024-03-03T00:00:00Z"
  },
  {
    _id: "p4",
    name: "Abaya Hitam Kancing Samping (Checker)",
    brand: "NVP EXCLUSIVE",
    price: 375000,
    originalPrice: 0,
    colors: ["Hitam"],
    sizes: ["S", "M", "L"],
    image: "photo_6204060867754659797_w.jpg",
    description: "Abaya premium dengan paduan aksen kemeja kotak-kotak (checker) di bagian samping dan lengan. Sangat stylish untuk hangout.",
    createdAt: "2024-03-04T00:00:00Z"
  },
  {
    _id: "p5",
    name: "Abaya Hitam Premium Checker Mix",
    brand: "NVP EXCLUSIVE",
    price: 375000,
    originalPrice: 420000,
    colors: ["Hitam"],
    sizes: ["M", "L", "XL"],
    image: "photo_6204060867754659798_w.jpg",
    description: "Kombinasi elegan abaya hitam polos dengan detail motif kotak-kotak. Terdapat kancing fungsional untuk memudahkan gaya Anda.",
    createdAt: "2024-03-05T00:00:00Z"
  },
  {
    _id: "p6",
    name: "Kintamani Silk Abaya",
    brand: "RIA MIRANDA",
    price: 1250000,
    originalPrice: 1500000,
    colors: ["Pink", "Coklat"],
    sizes: ["S", "M"],
    image: "https://images.unsplash.com/photo-1605763240000-7e93b172d754?auto=format&fit=crop&q=80&w=600&h=800",
    description: "Abaya premium dari koleksi eksklusif. Menggunakan bahan maxmara silk dengan motif print.",
    createdAt: "2023-09-15T00:00:00Z"
  },
  {
    _id: "p7",
    name: "Lace Trimmed Kaftan",
    brand: "KAMI",
    price: 899000,
    originalPrice: 0,
    colors: ["Putih", "Cream"],
    sizes: ["All Size"],
    image: "https://images.unsplash.com/photo-1583391733958-67520c5bf315?auto=format&fit=crop&q=80&w=600&h=800",
    description: "Kaftan cantik dengan pinggiran brukat premium. Dilengkapi inner senada.",
    createdAt: "2023-08-20T00:00:00Z"
  },
  {
    _id: "p8",
    name: "Tiered Flowy Dress",
    brand: "ZALIA",
    price: 459000,
    originalPrice: 599000,
    colors: ["Biru Muda", "Lilac"],
    sizes: ["S", "M", "L"],
    image: "https://images.unsplash.com/photo-1515347619152-c6bf2e53fa5a?auto=format&fit=crop&q=80&w=600&h=800",
    description: "Gamis model susun (tiered) yang memberikan efek jenjang dan flowy saat berjalan.",
    createdAt: "2023-10-12T00:00:00Z"
  }
];

// ==========================================
// 2. MOCK EXPRESS.JS API ENDPOINTS
// ==========================================
const api = {
  getProducts: async ({ search, brands, colors, sort, page = 1, limit = 8 }) => {
    return new Promise((resolve) => {
      setTimeout(() => {
        let results = [...GLOBAL_DB];

        if (search) {
          const q = search.toLowerCase();
          results = results.filter(p => p.name.toLowerCase().includes(q) || p.brand.toLowerCase().includes(q));
        }
        if (brands && brands.length > 0) {
          results = results.filter(p => brands.includes(p.brand));
        }
        if (colors && colors.length > 0) {
          results = results.filter(p => p.colors.some(c => colors.includes(c)));
        }
        if (sort === 'newest') {
          results.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
        } else if (sort === 'price-asc') {
          results.sort((a, b) => a.price - b.price);
        } else if (sort === 'price-desc') {
          results.sort((a, b) => b.price - a.price);
        }

        const total = results.length;
        const start = (page - 1) * limit;
        const paginatedResults = results.slice(0, start + limit);

        resolve({ data: paginatedResults, total, hasMore: start + limit < total });
      }, 300);
    });
  },
  getProductById: async (id) => {
    return new Promise((resolve) => {
      setTimeout(() => {
        const product = GLOBAL_DB.find(p => p._id === id);
        resolve({ data: product });
      }, 200);
    });
  }
};

// ==========================================
// 3. MAIN REACT APP
// ==========================================
export default function App() {
  const [currentView, setCurrentView] = useState('category'); // 'category', 'detail', 'cart'
  const [selectedProductId, setSelectedProductId] = useState(null);
  
  // App State
  const [cart, setCart] = useState([]);
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
  const [isAddProductModalOpen, setIsAddProductModalOpen] = useState(false);
  const [refreshTrigger, setRefreshTrigger] = useState(0);

  // Formatter
  const formatRupiah = (number) => {
    return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number);
  };

  // View Router
  const navigateTo = (view, id = null) => {
    setCurrentView(view);
    if (id) setSelectedProductId(id);
    window.scrollTo(0, 0);
  };

  // --- COMPONENT: Add Product Modal ---
  const AddProductModal = () => {
    if (!isAddProductModalOpen) return null;

    const handleSubmit = (e) => {
      e.preventDefault();
      const file = e.target.imageFile.files[0];
      
      // Mengubah file gambar menjadi local object URL agar bisa ditampilkan
      const imageUrl = file ? URL.createObjectURL(file) : "https://via.placeholder.com/600x800";

      const newProduct = {
        _id: "p_" + Date.now(),
        name: e.target.name.value,
        brand: e.target.brand.value || "NVP BRAND",
        price: parseInt(e.target.price.value),
        originalPrice: 0,
        colors: ["Varian Baru"],
        sizes: ["S", "M", "L", "XL"],
        image: imageUrl,
        description: e.target.description.value,
        createdAt: new Date().toISOString()
      };

      // Menambahkan ke Database Bawaan
      GLOBAL_DB.unshift(newProduct);
      setRefreshTrigger(prev => prev + 1); // Me-refresh katalog
      setIsAddProductModalOpen(false);
      alert("Produk berhasil ditambahkan!");
    };

    return (
      <div className="fixed inset-0 z-[100] bg-black bg-opacity-60 flex items-center justify-center p-4">
        <div className="bg-white rounded-lg shadow-xl max-w-lg w-full p-6 relative">
          <button 
            onClick={() => setIsAddProductModalOpen(false)}
            className="absolute top-4 right-4 text-gray-500 hover:text-black"
          >
            <X size={24} />
          </button>
          <h2 className="text-2xl font-bold mb-6">Tambah Produk Baru</h2>
          
          <form onSubmit={handleSubmit} className="space-y-4">
            <div>
              <label className="block text-sm font-bold text-gray-700 mb-1">Upload Foto Produk</label>
              <input 
                type="file" 
                name="imageFile" 
                accept="image/*" 
                required
                className="w-full border border-gray-300 p-2 rounded focus:outline-none focus:border-black"
              />
            </div>
            <div>
              <label className="block text-sm font-bold text-gray-700 mb-1">Nama Produk</label>
              <input 
                type="text" 
                name="name" 
                required placeholder="Contoh: Abaya Hitam Premium"
                className="w-full border border-gray-300 p-2 rounded focus:outline-none focus:border-black"
              />
            </div>
            <div className="grid grid-cols-2 gap-4">
              <div>
                <label className="block text-sm font-bold text-gray-700 mb-1">Brand</label>
                <input 
                  type="text" 
                  name="brand" 
                  required placeholder="NVP EXCLUSIVE"
                  className="w-full border border-gray-300 p-2 rounded focus:outline-none focus:border-black"
                />
              </div>
              <div>
                <label className="block text-sm font-bold text-gray-700 mb-1">Harga (Rp)</label>
                <input 
                  type="number" 
                  name="price" 
                  required placeholder="350000"
                  className="w-full border border-gray-300 p-2 rounded focus:outline-none focus:border-black"
                />
              </div>
            </div>
            <div>
              <label className="block text-sm font-bold text-gray-700 mb-1">Deskripsi</label>
              <textarea 
                name="description" 
                required placeholder="Tulis deskripsi produk di sini..."
                rows="3"
                className="w-full border border-gray-300 p-2 rounded focus:outline-none focus:border-black"
              ></textarea>
            </div>
            <button 
              type="submit"
              className="w-full bg-black text-white font-bold py-3 uppercase hover:bg-gray-800 transition-colors mt-4"
            >
              Simpan Produk
            </button>
          </form>
        </div>
      </div>
    );
  };

  // --- COMPONENT: Navbar ---
  const Navbar = () => (
    <nav className="sticky top-0 z-50 bg-white border-b border-gray-200">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between items-center h-16">
          <div className="flex items-center">
            <button className="sm:hidden p-2 -ml-2 text-gray-600" onClick={() => setIsMobileMenuOpen(true)}>
              <Menu size={24} />
            </button>
            <h1 
              className="text-2xl font-bold tracking-widest cursor-pointer ml-2 sm:ml-0" 
              onClick={() => navigateTo('category')}
            >
              NVP PROJECT
            </h1>
            <div className="hidden sm:flex ml-10 space-x-6 items-center">
              <a href="#" className="text-sm font-medium text-gray-900 border-b-2 border-black pb-1">WANITA</a>
              <a href="#" className="text-sm font-medium text-gray-500 hover:text-gray-900">PRIA</a>
              <a href="#" className="text-sm font-medium text-gray-500 hover:text-gray-900">ANAK</a>
              {/* Tombol Tambah Produk */}
              <button 
                onClick={() => setIsAddProductModalOpen(true)}
                className="ml-4 flex items-center text-xs font-bold bg-gray-100 px-3 py-1.5 rounded-full hover:bg-gray-200 transition-colors"
              >
                <Plus size={14} className="mr-1"/> Tambah Produk
              </button>
            </div>
          </div>
          <div className="flex items-center space-x-4 sm:space-x-6">
            <div className="hidden sm:block relative">
              <input 
                type="text" 
                placeholder="Cari produk..." 
                className="w-48 lg:w-64 bg-gray-100 border-none rounded-md py-2 pl-4 pr-10 text-sm focus:ring-1 focus:ring-black focus:bg-white transition-all"
              />
              <Search className="absolute right-3 top-2 text-gray-400" size={18} />
            </div>
            <button className="sm:hidden text-gray-600">
              <Search size={24} />
            </button>
            <button 
              className="text-gray-600 relative p-2 -mr-2"
              onClick={() => navigateTo('cart')}
            >
              <ShoppingBag size={24} />
              {cart.length > 0 && (
                <span className="absolute top-0 right-0 inline-flex items-center justify-center px-1.5 py-0.5 text-xs font-bold leading-none text-white transform translate-x-1/4 -translate-y-1/4 bg-red-600 rounded-full">
                  {cart.length}
                </span>
              )}
            </button>
          </div>
        </div>
      </div>
    </nav>
  );

  // --- COMPONENT: Category Page ---
  const CategoryPage = () => {
    const [products, setProducts] = useState([]);
    const [loading, setLoading] = useState(true);
    const [page, setPage] = useState(1);
    const [hasMore, setHasMore] = useState(true);
    
    const [sort, setSort] = useState('newest');
    const [selectedBrands, setSelectedBrands] = useState([]);
    const [isFilterMobileOpen, setIsFilterMobileOpen] = useState(false);

    const availableBrands = [...new Set(GLOBAL_DB.map(p => p.brand))].sort();

    const fetchProductsData = async (currentPage, isLoadMore = false) => {
      if (!isLoadMore) setLoading(true);
      const res = await api.getProducts({
        brands: selectedBrands,
        sort: sort,
        page: currentPage,
        limit: 8
      });
      if (isLoadMore) {
        setProducts(res.data);
      } else {
        setProducts(res.data);
      }
      setHasMore(res.hasMore);
      setLoading(false);
    };

    useEffect(() => {
      setPage(1);
      fetchProductsData(1, false);
    }, [selectedBrands, sort, refreshTrigger]); // Akan ter-refresh jika refreshTrigger berubah

    const handleLoadMore = () => {
      const nextPage = page + 1;
      setPage(nextPage);
      fetchProductsData(nextPage, true);
    };

    const toggleFilter = (brand) => {
      setSelectedBrands(prev => prev.includes(brand) ? prev.filter(i => i !== brand) : [...prev, brand]);
    };

    return (
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        {/* Mobile Header Tombol Tambah Produk */}
        <div className="sm:hidden mb-6 flex justify-between items-center bg-gray-50 p-3 rounded-lg border border-gray-200">
           <span className="text-sm font-medium">Mode Admin:</span>
           <button 
              onClick={() => setIsAddProductModalOpen(true)}
              className="flex items-center text-xs font-bold bg-black text-white px-3 py-2 rounded shadow-sm"
            >
              <Plus size={14} className="mr-1"/> Tambah Produk
            </button>
        </div>

        <div className="text-sm text-gray-500 mb-4">
          Wanita / Pakaian / <span className="text-gray-900">Gamis & Abaya</span>
        </div>
        <h2 className="text-2xl font-bold mb-8 uppercase">Koleksi Abaya NVP</h2>

        <div className="flex flex-col md:flex-row gap-8">
          <div className="flex justify-between items-center md:hidden mb-4">
            <button 
              onClick={() => setIsFilterMobileOpen(!isFilterMobileOpen)}
              className="flex items-center text-sm font-medium py-2 px-4 border border-gray-300 rounded"
            >
              <Filter size={16} className="mr-2" /> Filter & Sort
            </button>
            <span className="text-sm text-gray-500">{products.length} produk</span>
          </div>

          <div className={`w-full md:w-64 flex-shrink-0 ${isFilterMobileOpen ? 'block' : 'hidden md:block'}`}>
            <div className="space-y-6">
              <div className="md:hidden">
                <h3 className="text-sm font-bold uppercase mb-3">Urutkan</h3>
                <select 
                  className="w-full border border-gray-300 text-sm focus:ring-black focus:border-black rounded py-2"
                  value={sort}
                  onChange={(e) => setSort(e.target.value)}
                >
                  <option value="newest">Terbaru</option>
                  <option value="price-asc">Harga Terendah</option>
                  <option value="price-desc">Harga Tertinggi</option>
                </select>
              </div>

              <div className="border-t border-gray-200 pt-6 md:border-none md:pt-0">
                <h3 className="text-sm font-bold uppercase mb-3 flex justify-between">Merek</h3>
                <div className="space-y-2 max-h-48 overflow-y-auto pr-2">
                  {availableBrands.map(brand => (
                    <label key={brand} className="flex items-center cursor-pointer group">
                      <div className="relative flex items-center justify-center w-4 h-4 mr-3 border border-gray-300 rounded-sm group-hover:border-black">
                        <input 
                          type="checkbox" 
                          className="opacity-0 absolute w-full h-full cursor-pointer"
                          checked={selectedBrands.includes(brand)}
                          onChange={() => toggleFilter(brand)}
                        />
                        {selectedBrands.includes(brand) && <Check size={12} className="text-black" />}
                      </div>
                      <span className="text-sm text-gray-600">{brand}</span>
                    </label>
                  ))}
                </div>
              </div>
            </div>
          </div>

          <div className="flex-1">
            <div className="hidden md:flex justify-between items-center mb-6">
              <span className="text-sm text-gray-500">{products.length} produk ditemukan</span>
              <div className="flex items-center">
                <span className="text-sm mr-2 text-gray-500">Urutkan:</span>
                <select 
                  className="border border-gray-300 text-sm focus:ring-black focus:border-black rounded-none py-1.5 pl-3 pr-8"
                  value={sort}
                  onChange={(e) => setSort(e.target.value)}
                >
                  <option value="newest">Terbaru</option>
                  <option value="price-asc">Harga Terendah</option>
                  <option value="price-desc">Harga Tertinggi</option>
                </select>
              </div>
            </div>

            {loading && page === 1 ? (
              <div className="flex justify-center items-center h-64">
                <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-black"></div>
              </div>
            ) : products.length === 0 ? (
              <div className="text-center py-16 text-gray-500">
                <p>Tidak ada produk yang sesuai dengan filter.</p>
                <button onClick={() => setSelectedBrands([])} className="mt-4 text-black underline font-medium">Reset Filter</button>
              </div>
            ) : (
              <>
                <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-x-4 gap-y-8 sm:gap-x-6 sm:gap-y-10">
                  {products.map(product => (
                    <div 
                      key={product._id} 
                      className="group cursor-pointer flex flex-col"
                      onClick={() => navigateTo('detail', product._id)}
                    >
                      <div className="relative aspect-[3/4] overflow-hidden bg-gray-100 mb-3">
                        <img 
                          src={product.image} 
                          alt={product.name}
                          className="object-cover w-full h-full group-hover:scale-105 transition-transform duration-500"
                        />
                        {product.originalPrice > 0 && (
                          <div className="absolute top-2 left-2 bg-red-600 text-white text-[10px] font-bold px-2 py-1 uppercase tracking-wider">
                            Sale
                          </div>
                        )}
                      </div>
                      <div className="flex-1 flex flex-col">
                        <h3 className="text-xs font-bold uppercase tracking-wider text-gray-900 mb-1">{product.brand}</h3>
                        <p className="text-sm text-gray-600 mb-2 line-clamp-2 leading-snug flex-1">{product.name}</p>
                        <div className="flex items-baseline space-x-2">
                          <span className={`text-sm font-bold ${product.originalPrice > 0 ? 'text-red-600' : 'text-gray-900'}`}>
                            {formatRupiah(product.price)}
                          </span>
                          {product.originalPrice > 0 && (
                            <span className="text-xs text-gray-400 line-through">
                              {formatRupiah(product.originalPrice)}
                            </span>
                          )}
                        </div>
                      </div>
                    </div>
                  ))}
                </div>
                
                {hasMore && (
                  <div className="mt-12 text-center border-t border-gray-200 pt-8">
                    <button 
                      onClick={handleLoadMore}
                      disabled={loading}
                      className="bg-white text-black border border-black font-bold uppercase text-sm px-16 py-3 hover:bg-black hover:text-white transition-colors disabled:opacity-50"
                    >
                      {loading ? 'Memuat...' : 'Muat Lebih Banyak'}
                    </button>
                  </div>
                )}
              </>
            )}
          </div>
        </div>
      </div>
    );
  };

  // --- COMPONENT: Product Detail ---
  const ProductDetail = ({ id }) => {
    const [product, setProduct] = useState(null);
    const [loading, setLoading] = useState(true);
    const [selectedSize, setSelectedSize] = useState('');
    const [errorMsg, setErrorMsg] = useState('');

    useEffect(() => {
      const fetchDetail = async () => {
        setLoading(true);
        const res = await api.getProductById(id);
        setProduct(res.data);
        setLoading(false);
      };
      fetchDetail();
    }, [id]);

    const handleAddToCart = () => {
      if (!selectedSize) {
        setErrorMsg('Pilih ukuran terlebih dahulu');
        return;
      }
      setErrorMsg('');
      setCart(prev => {
        const existing = prev.find(item => item.product._id === product._id && item.size === selectedSize);
        if (existing) {
          return prev.map(item => item === existing ? { ...item, quantity: item.quantity + 1 } : item);
        }
        return [...prev, { product, size: selectedSize, quantity: 1 }];
      });
      alert('Produk berhasil ditambahkan ke keranjang!');
    };

    if (loading) return <div className="min-h-screen flex justify-center items-center"><div className="animate-spin rounded-full h-8 w-8 border-b-2 border-black"></div></div>;
    if (!product) return <div className="p-8 text-center">Produk tidak ditemukan.</div>;

    return (
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <button 
          onClick={() => navigateTo('category')}
          className="flex items-center text-sm text-gray-500 hover:text-black mb-6"
        >
          <ChevronLeft size={16} className="mr-1" /> Kembali ke Katalog
        </button>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-10">
          <div className="bg-gray-100 aspect-[3/4]">
            <img src={product.image} alt={product.name} className="w-full h-full object-cover" />
          </div>

          <div className="flex flex-col py-4">
            <h1 className="text-xl font-bold uppercase tracking-widest text-gray-900 mb-2">{product.brand}</h1>
            <h2 className="text-lg text-gray-600 mb-6">{product.name}</h2>
            
            <div className="flex items-center space-x-3 mb-8">
              <span className="text-2xl font-bold text-gray-900">{formatRupiah(product.price)}</span>
              {product.originalPrice > 0 && (
                <span className="text-lg text-gray-400 line-through">{formatRupiah(product.originalPrice)}</span>
              )}
            </div>

            <div className="mb-6">
              <div className="flex justify-between mb-2">
                <span className="text-sm font-bold uppercase">Pilih Ukuran</span>
              </div>
              <div className="flex flex-wrap gap-3">
                {product.sizes.map(size => (
                  <button
                    key={size}
                    onClick={() => { setSelectedSize(size); setErrorMsg(''); }}
                    className={`min-w-[3rem] h-12 px-4 flex items-center justify-center border text-sm font-medium transition-colors
                      ${selectedSize === size 
                        ? 'border-black bg-black text-white' 
                        : 'border-gray-300 text-gray-900 hover:border-black'}`}
                  >
                    {size}
                  </button>
                ))}
              </div>
              {errorMsg && <p className="text-red-500 text-xs mt-2 font-medium">{errorMsg}</p>}
            </div>

            <button 
              onClick={handleAddToCart}
              className="w-full bg-black text-white uppercase font-bold text-sm py-4 mb-8 hover:bg-gray-800 transition-colors shadow-lg shadow-gray-300"
            >
              Tambah ke Keranjang
            </button>

            <div className="border-t border-gray-200 pt-6">
              <h3 className="text-sm font-bold uppercase mb-3">Deskripsi Produk</h3>
              <p className="text-sm text-gray-600 leading-relaxed whitespace-pre-line">
                {product.description}
              </p>
            </div>
          </div>
        </div>
      </div>
    );
  };

  // --- COMPONENT: Cart ---
  const CartPage = () => {
    const total = cart.reduce((sum, item) => sum + (item.product.price * item.quantity), 0);

    const updateQty = (index, delta) => {
      const newCart = [...cart];
      newCart[index].quantity += delta;
      if (newCart[index].quantity <= 0) {
        newCart.splice(index, 1);
      }
      setCart(newCart);
    };

    const removeItem = (index) => {
      const newCart = [...cart];
      newCart.splice(index, 1);
      setCart(newCart);
    };

    const handleCheckout = () => {
      const phoneNumber = "6281234567890"; // Nomor WA Admin
      let message = "Halo admin NVP Project, saya ingin memesan:\n\n";
      
      cart.forEach((item, index) => {
        message += `${index + 1}. ${item.product.brand} - ${item.product.name}\n`;
        message += `   Ukuran: ${item.size}\n`;
        message += `   Jumlah: ${item.quantity}\n`;
        message += `   Harga: ${formatRupiah(item.product.price * item.quantity)}\n\n`;
      });

      message += `*Total Pesanan: ${formatRupiah(total)}*\n\n`;
      message += "Mohon info total tagihan beserta ongkos kirimnya ya. Terima kasih!";

      const encodedMessage = encodeURIComponent(message);
      window.open(`https://wa.me/${phoneNumber}?text=${encodedMessage}`, '_blank');
    };

    return (
      <div className="max-w-4xl mx-auto px-4 py-8">
        <h2 className="text-2xl font-bold uppercase tracking-wider border-b border-gray-200 pb-4 mb-6">Tas Belanja Saya ({cart.length})</h2>
        
        {cart.length === 0 ? (
          <div className="text-center py-16">
            <div className="bg-gray-100 w-24 h-24 rounded-full flex items-center justify-center mx-auto mb-4">
              <ShoppingBag size={40} className="text-gray-400" />
            </div>
            <h3 className="text-xl font-medium mb-2">Tas belanja Anda masih kosong</h3>
            <p className="text-gray-500 mb-8">Temukan banyak pilihan fashion menarik!</p>
            <button 
              onClick={() => navigateTo('category')}
              className="bg-black text-white px-8 py-3 text-sm font-bold uppercase hover:bg-gray-800"
            >
              Mulai Belanja
            </button>
          </div>
        ) : (
          <div className="flex flex-col md:flex-row gap-8">
            <div className="flex-1 space-y-6">
              {cart.map((item, idx) => (
                <div key={`${item.product._id}-${item.size}`} className="flex gap-4 border border-gray-200 p-4 relative shadow-sm hover:shadow-md transition-shadow">
                  <button 
                    onClick={() => removeItem(idx)}
                    className="absolute top-4 right-4 text-gray-400 hover:text-black"
                  >
                    <X size={18} />
                  </button>
                  <img src={item.product.image} alt={item.product.name} className="w-24 h-32 object-cover bg-gray-100" />
                  <div className="flex-1 flex flex-col justify-between">
                    <div>
                      <h4 className="font-bold text-sm uppercase">{item.product.brand}</h4>
                      <p className="text-sm text-gray-600 line-clamp-1">{item.product.name}</p>
                      <p className="text-sm text-gray-500 mt-1">Ukuran: {item.size}</p>
                    </div>
                    <div className="flex justify-between items-end mt-4">
                      <div className="flex items-center border border-gray-300">
                        <button onClick={() => updateQty(idx, -1)} className="px-3 py-1 text-gray-600 hover:bg-gray-100">-</button>
                        <span className="px-3 py-1 text-sm text-center min-w-[2rem] border-x border-gray-300">{item.quantity}</span>
                        <button onClick={() => updateQty(idx, 1)} className="px-3 py-1 text-gray-600 hover:bg-gray-100">+</button>
                      </div>
                      <span className="font-bold text-gray-900">{formatRupiah(item.product.price * item.quantity)}</span>
                    </div>
                  </div>
                </div>
              ))}
            </div>
            
            <div className="w-full md:w-80 flex-shrink-0">
              <div className="bg-gray-50 border border-gray-200 p-6 sticky top-24">
                <h3 className="font-bold uppercase mb-4 pb-4 border-b border-gray-200">Ringkasan Pesanan</h3>
                <div className="flex justify-between text-sm mb-3">
                  <span className="text-gray-600">Subtotal</span>
                  <span className="font-medium text-gray-900">{formatRupiah(total)}</span>
                </div>
                <div className="flex justify-between text-sm mb-6">
                  <span className="text-gray-600">Estimasi Pengiriman</span>
                  <span className="font-medium text-green-600">Gratis</span>
                </div>
                <div className="flex justify-between text-base font-bold mb-8 pt-4 border-t border-gray-200">
                  <span>Total</span>
                  <span>{formatRupiah(total)}</span>
                </div>
                <button 
                  onClick={handleCheckout}
                  className="w-full bg-black text-white uppercase font-bold text-sm py-4 hover:bg-gray-800 transition-colors shadow-lg"
                >
                  Checkout via WhatsApp
                </button>
              </div>
            </div>
          </div>
        )}
      </div>
    );
  };

  return (
    <div className="min-h-screen bg-white font-sans text-gray-900 relative">
      <div className="bg-black text-white text-xs text-center py-2 uppercase tracking-wide">
        Gratis ongkir untuk pesanan di atas Rp 300.000!
      </div>
      
      <Navbar />

      <main>
        {currentView === 'category' && <CategoryPage />}
        {currentView === 'detail' && <ProductDetail id={selectedProductId} />}
        {currentView === 'cart' && <CartPage />}
      </main>

      <footer className="bg-gray-100 mt-20 py-12 border-t border-gray-200">
        <div className="max-w-7xl mx-auto px-4 text-center text-sm text-gray-500">
          <p className="font-bold tracking-widest text-black mb-4">NVP PROJECT</p>
          <p>© 2026 E-Commerce Demo. Dibuat menggunakan React & Tailwind CSS.</p>
        </div>
      </footer>

      {/* Menampilkan Modal Tambah Produk (di atas layar) */}
      <AddProductModal />
    </div>
  );
}
