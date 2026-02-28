import React, { useState, useMemo, useRef } from 'react';
import { 
  Plus, 
  Search, 
  AlertTriangle, 
  Package, 
  Camera, 
  Trash2, 
  Edit3, 
  X, 
  Check, 
  AlertCircle,
  MinusCircle,
  PlusCircle,
  ArrowRight
} from 'lucide-react';

const App = () => {
  // 狀態管理：庫存列表 (預設兩筆示範資料)
  const [inventory, setInventory] = useState([
    {
      id: 1,
      name: "示範商品 A",
      sku: "SKU-001",
      stock: 5,
      warningLevel: 10,
      image: null,
      updatedAt: new Date().toLocaleDateString()
    },
    {
      id: 2,
      name: "示範商品 B",
      sku: "SKU-002",
      stock: 25,
      warningLevel: 15,
      image: null,
      updatedAt: new Date().toLocaleDateString()
    }
  ]);

  // UI 狀態
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [editingItem, setEditingItem] = useState(null);
  const [searchTerm, setSearchTerm] = useState('');
  const [filterLowStock, setFilterLowStock] = useState(false);
  const [notification, setNotification] = useState(null);
  const [deleteTarget, setDeleteTarget] = useState(null);
  
  // 引用隱藏的檔案輸入框
  const fileInputRef = useRef(null);

  // 表單狀態
  const [formData, setFormData] = useState({
    name: '',
    sku: '',
    stock: 0,
    warningLevel: 5,
    image: null
  });

  // 顯示通知
  const showNotice = (msg) => {
    setNotification(msg);
    setTimeout(() => setNotification(null), 3000);
  };

  // 處理照片選取 (修正後的版本)
  const handleImageChange = (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setFormData({ ...formData, image: reader.result });
      };
      reader.readAsDataURL(file);
    }
  };

  // 開啟視窗
  const openModal = (item = null) => {
    if (item) {
      setEditingItem(item);
      setFormData(item);
    } else {
      setEditingItem(null);
      setFormData({ name: '', sku: '', stock: 0, warningLevel: 5, image: null });
    }
    setIsModalOpen(true);
  };

  // 儲存邏輯
  const handleSubmit = (e) => {
    e.preventDefault();
    const updatedData = { 
      ...formData, 
      updatedAt: new Date().toLocaleDateString(),
      stock: Number(formData.stock),
      warningLevel: Number(formData.warningLevel)
    };

    if (editingItem) {
      setInventory(inventory.map(item => 
        item.id === editingItem.id ? { ...updatedData, id: item.id } : item
      ));
      showNotice("商品資料已更新");
    } else {
      setInventory([...inventory, { ...updatedData, id: Date.now() }]);
      showNotice("新商品已新增");
    }
    setIsModalOpen(false);
  };

  // 確認刪除
  const confirmDelete = () => {
    if (deleteTarget) {
      setInventory(inventory.filter(item => item.id !== deleteTarget.id));
      showNotice(`已移除商品：${deleteTarget.name}`);
      setDeleteTarget(null);
    }
  };

  // 篩選邏輯
  const filteredInventory = useMemo(() => {
    return inventory.filter(item => {
      const name = item.name || '';
      const sku = item.sku || '';
      const matchesSearch = name.toLowerCase().includes(searchTerm.toLowerCase()) || 
                            sku.toLowerCase().includes(searchTerm.toLowerCase());
      const matchesFilter = filterLowStock ? item.stock <= item.warningLevel : true;
      return matchesSearch && matchesFilter;
    });
  }, [inventory, searchTerm, filterLowStock]);

  const lowStockCount = inventory.filter(i => i.stock <= i.warningLevel).length;

  return (
    <div className="min-h-screen bg-emerald-50/50 text-slate-900 font-sans">
      {/* 導航欄 */}
      <nav className="bg-white border-b border-emerald-100 sticky top-0 z-40 px-4 py-3 flex justify-between items-center shadow-sm">
        <div className="flex items-center gap-2">
          <div className="bg-emerald-500 p-2 rounded-lg">
            <Package className="text-white w-5 h-5" />
          </div>
          <h1 className="font-black text-xl tracking-tight text-emerald-900">庫存管家</h1>
        </div>
        <button 
          onClick={() => openModal()}
          className="bg-emerald-600 hover:bg-emerald-700 text-white px-5 py-2 rounded-full flex items-center gap-2 transition-all shadow-md active:scale-95 font-bold"
        >
          <Plus size={20} />
          <span>新增商品</span>
        </button>
      </nav>

      <main className="max-w-6xl mx-auto p-4 sm:p-8">
        {/* 狀態板 */}
        <div className="grid grid-cols-1 sm:grid-cols-3 gap-4 mb-8">
          <div className="bg-white p-5 rounded-2xl shadow-sm border border-emerald-100">
            <p className="text-emerald-600/70 text-sm mb-1 font-bold">總品項</p>
            <p className="text-3xl font-black text-emerald-900">{inventory.length}</p>
          </div>
          <div className={`p-5 rounded-2xl shadow-sm border transition-all ${lowStockCount > 0 ? 'bg-red-50 border-red-200 animate-pulse' : 'bg-white border-emerald-100'}`}>
            <div className="flex justify-between items-start">
              <div>
                <p className={`${lowStockCount > 0 ? 'text-red-700' : 'text-emerald-600/70'} text-sm mb-1 font-bold`}>庫存警示</p>
                <p className={`text-3xl font-black ${lowStockCount > 0 ? 'text-red-600' : 'text-emerald-900'}`}>{lowStockCount}</p>
              </div>
              {lowStockCount > 0 && <AlertTriangle className="text-red-500 w-8 h-8" />}
            </div>
          </div>
          <div className="bg-white p-5 rounded-2xl shadow-sm border border-emerald-100 flex items-end">
            <div className="w-full relative">
              <Search className="absolute left-3 top-3 text-emerald-400 w-4 h-4" />
              <input 
                type="text" placeholder="搜尋名稱或 SKU..." 
                className="w-full pl-9 pr-4 py-2 bg-emerald-50/50 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 border border-emerald-100 font-medium"
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
              />
            </div>
          </div>
        </div>

        {/* 篩選器 */}
        <div className="flex items-center gap-2 mb-6">
          <button 
            onClick={() => setFilterLowStock(false)}
            className={`px-6 py-2 rounded-full text-sm font-bold transition-all ${!filterLowStock ? 'bg-emerald-600 text-white shadow-lg' : 'bg-white text-emerald-700 border border-emerald-200'}`}
          >
            全部商品
          </button>
          <button 
            onClick={() => setFilterLowStock(true)}
            className={`px-6 py-2 rounded-full text-sm font-bold flex items-center gap-1 transition-all ${filterLowStock ? 'bg-red-600 text-white shadow-lg' : 'bg-white text-red-600 border border-red-200'}`}
          >
            <AlertTriangle size={14} />
            庫存不足
          </button>
        </div>

        {/* 清單表格 */}
        <div className="bg-white rounded-3xl shadow-xl border border-emerald-100 overflow-hidden">
          <div className="overflow-x-auto">
            <table className="w-full text-left">
              <thead>
                <tr className="bg-emerald-50/50 text-emerald-800 text-sm uppercase tracking-wider">
                  <th className="px-6 py-5 font-black">商品資訊</th>
                  <th className="px-6 py-5 font-black">SKU</th>
                  <th className="px-6 py-5 font-black">庫存量</th>
                  <th className="px-6 py-5 font-black text-center">操作</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-emerald-50">
                {filteredInventory.map((item) => (
                  <tr key={item.id} className="hover:bg-emerald-50/30 transition-colors group">
                    <td className="px-6 py-4">
                      <div className="flex items-center gap-4">
                        <div className="w-14 h-14 rounded-xl bg-emerald-50 overflow-hidden flex-shrink-0 border border-emerald-100 flex items-center justify-center">
                          {item.image ? (
                            <img src={item.image} alt={item.name} className="w-full h-full object-cover" />
                          ) : (
                            <Package className="text-emerald-200 w-7 h-7" />
                          )}
                        </div>
                        <div>
                          <p className="font-bold text-slate-800 text-lg">{item.name}</p>
                          <p className="text-[11px] text-slate-400 font-medium">更新：{item.updatedAt}</p>
                        </div>
                      </div>
                    </td>
                    <td className="px-6 py-4">
                      <span className="text-xs bg-emerald-50 px-3 py-1.5 rounded-lg text-emerald-700 font-bold font-mono uppercase tracking-wider border border-emerald-100">
                        {item.sku}
                      </span>
                    </td>
                    <td className="px-6 py-4">
                      <div className="flex flex-col">
                        <span className={`text-2xl font-black ${item.stock <= item.warningLevel ? 'text-red-600 flex items-center gap-1' : 'text-emerald-800'}`}>
                          {item.stock}
                          {item.stock <= item.warningLevel && <AlertTriangle size={18} className="animate-pulse" />}
                        </span>
                        <span className="text-[10px] text-slate-400 font-bold uppercase tracking-tighter">低於 {item.warningLevel} 警報</span>
                      </div>
                    </td>
                    <td className="px-6 py-4 text-center">
                      <div className="flex items-center justify-center gap-2 sm:opacity-0 group-hover:opacity-100 transition-opacity">
                        <button onClick={() => openModal(item)} className="p-2.5 hover:bg-emerald-100 text-emerald-600 rounded-xl transition-all shadow-sm"><Edit3 size={18} /></button>
                        <button onClick={() => setDeleteTarget(item)} className="p-2.5 hover:bg-red-100 text-red-600 rounded-xl transition-all shadow-sm"><Trash2 size={18} /></button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
          {filteredInventory.length === 0 && (
            <div className="py-20 text-center flex flex-col items-center gap-4">
              <Package size={64} className="text-emerald-100 opacity-50" />
              <p className="text-emerald-800/40 font-bold text-xl tracking-widest">目前沒有任何商品項目</p>
            </div>
          )}
        </div>
      </main>

      {/* 刪除確認對話框 */}
      {deleteTarget && (
        <div className="fixed inset-0 bg-emerald-950/40 backdrop-blur-sm z-[60] flex items-center justify-center p-4">
          <div className="bg-white w-full max-w-sm rounded-[2.5rem] p-8 text-center animate-in zoom-in shadow-2xl border-4 border-white">
            <div className="w-20 h-20 bg-red-50 text-red-500 rounded-full flex items-center justify-center mx-auto mb-6"><Trash2 size={40} /></div>
            <h3 className="text-2xl font-black text-slate-800 mb-2">確定要刪除？</h3>
            <p className="text-slate-500 mb-8 font-medium">「{deleteTarget.name}」將永久從庫存消失。</p>
            <div className="flex gap-4">
              <button onClick={() => setDeleteTarget(null)} className="flex-1 py-4 bg-slate-100 text-slate-600 font-bold rounded-2xl active:scale-95 transition-transform">取消</button>
              <button onClick={confirmDelete} className="flex-1 py-4 bg-red-500 text-white font-bold rounded-2xl shadow-lg shadow-red-200 active:scale-95 transition-transform">確定刪除</button>
            </div>
          </div>
        </div>
      )}

      {/* 新增/編輯視窗 (包含字體加大與寬欄位優化) */}
      {isModalOpen && (
        <div className="fixed inset-0 bg-emerald-950/30 backdrop-blur-xl z-50 flex items-center justify-center p-4 overflow-y-auto">
          <div className="bg-white w-full max-w-xl rounded-[3.5rem] shadow-2xl overflow-hidden animate-in slide-in-from-bottom duration-300 my-auto border-4 border-white">
            <div className="px-10 py-8 border-b border-emerald-50 flex justify-between items-center bg-emerald-50/40">
              <h2 className="text-2xl font-black text-emerald-900">{editingItem ? '調整商品資料' : '新商品入庫'}</h2>
              <button onClick={() => setIsModalOpen(false)} className="p-3 hover:bg-white rounded-full transition-colors shadow-sm"><X size={24} className="text-emerald-900" /></button>
            </div>
            
            <form onSubmit={handleSubmit} className="p-10 space-y-8">
              {/* 照片上傳區域 */}
              <div className="flex flex-col sm:flex-row gap-8 items-center">
                <div 
                  onClick={() => fileInputRef.current.click()}
                  className="relative w-44 h-44 rounded-[2.5rem] bg-emerald-50 border-4 border-dashed border-emerald-200 shadow-inner overflow-hidden flex items-center justify-center group cursor-pointer transition-all hover:border-emerald-400 active:scale-95 flex-shrink-0"
                >
                  {formData.image ? (
                    <img src={formData.image} alt="Preview" className="w-full h-full object-cover" />
                  ) : (
                    <div className="text-center text-emerald-400 space-y-2">
                      <Camera size={48} className="mx-auto" />
                      <span className="text-sm font-black uppercase block tracking-widest">拍照/上傳</span>
                    </div>
                  )}
                  <input ref={fileInputRef} type="file" accept="image/*" capture="environment" onChange={handleImageChange} className="hidden" />
                </div>
                
                <div className="flex-1 space-y-6 w-full">
                  <div>
                    <label className="text-lg font-black text-emerald-800 block mb-2 uppercase tracking-wide">商品名稱</label>
                    <input required placeholder="輸入商品名稱..." className="w-full px-6 py-4 bg-slate-50 border-2 border-transparent focus:border-emerald-500 focus:bg-white rounded-3xl outline-none font-bold text-xl transition-all shadow-sm" value={formData.name} onChange={(e) => setFormData({...formData, name: e.target.value})} />
                  </div>
                  <div>
                    <label className="text-lg font-black text-emerald-800 block mb-2 uppercase tracking-wide">SKU 編號</label>
                    <input required placeholder="SKU-XXXX" className="w-full px-6 py-4 bg-slate-50 border-2 border-transparent focus:border-emerald-500 focus:bg-white rounded-3xl outline-none font-mono text-lg uppercase transition-all shadow-sm" value={formData.sku} onChange={(e) => setFormData({...formData, sku: e.target.value.toUpperCase()})} />
                  </div>
                </div>
              </div>

              {/* 庫存與警示欄位 (加寬卡片) */}
              <div className="grid grid-cols-1 sm:grid-cols-2 gap-6">
                <div className="bg-emerald-50/50 p-8 rounded-[3rem] border-2 border-emerald-100 flex flex-col items-center">
                  <label className="text-base font-black text-emerald-700 block mb-4 uppercase tracking-widest bg-emerald-100/50 px-6 py-1.5 rounded-full">當前庫存</label>
                  <div className="flex items-center justify-center gap-6 w-full">
                    <button type="button" onClick={() => setFormData({...formData, stock: Math.max(0, Number(formData.stock) - 1)})} className="text-emerald-500 hover:text-emerald-700 active:scale-75 transition-transform"><MinusCircle size={48} /></button>
                    <input type="number" className="w-28 text-center bg-white border-2 border-emerald-100 rounded-2xl py-3 text-4xl font-black text-emerald-900 outline-none shadow-sm focus:border-emerald-400" value={formData.stock} onChange={(e) => setFormData({...formData, stock: e.target.value})} />
                    <button type="button" onClick={() => setFormData({...formData, stock: Number(formData.stock) + 1})} className="text-emerald-500 hover:text-emerald-700 active:scale-75 transition-transform"><PlusCircle size={48} /></button>
                  </div>
                </div>
                <div className="bg-red-50/40 p-8 rounded-[3rem] border-2 border-red-100 flex flex-col items-center">
                  <label className="text-base font-black text-red-700 block mb-4 uppercase tracking-widest bg-red-100/50 px-6 py-1.5 rounded-full">警示水位</label>
                  <div className="flex items-center justify-center gap-6 w-full">
                    <button type="button" onClick={() => setFormData({...formData, warningLevel: Math.max(0, Number(formData.warningLevel) - 1)})} className="text-red-400 hover:text-red-600 active:scale-75 transition-transform"><MinusCircle size={48} /></button>
                    <input type="number" className="w-28 text-center bg-white border-2 border-red-100 rounded-2xl py-3 text-4xl font-black text-red-700 outline-none shadow-sm focus:border-red-400" value={formData.warningLevel} onChange={(e) => setFormData({...formData, warningLevel: e.target.value})} />
                    <button type="button" onClick={() => setFormData({...formData, warningLevel: Number(formData.warningLevel) + 1})} className="text-red-400 hover:text-red-600 active:scale-75 transition-transform"><PlusCircle size={48} /></button>
                  </div>
                </div>
              </div>

              <button type="submit" className="w-full py-6 bg-emerald-600 text-white rounded-[2.5rem] font-black text-xl shadow-xl shadow-emerald-100 flex items-center justify-center gap-3 transition-all active:scale-95 hover:bg-emerald-700">
                <Check size={28} /> 確認儲存
              </button>
            </form>
          </div>
        </div>
      )}

      {/* 提示通知 */}
      {notification && (
        <div className="fixed bottom-10 left-1/2 -translate-x-1/2 bg-slate-900/95 text-white px-10 py-5 rounded-full shadow-2xl flex items-center gap-4 animate-in fade-in slide-in-from-bottom-6 z-[100] border border-white/10">
          <div className="bg-emerald-500 rounded-full p-1.5"><Check size={20} className="text-white" /></div>
          <span className="font-black text-lg tracking-wide">{notification}</span>
        </div>
      )}
    </div>
  );
};

export default App;
