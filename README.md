import React, { useState, useEffect } from 'react';
import { 
  Bell, Camera, ShieldCheck, Heart, AlertTriangle, Plus, Trash2, CheckCircle2, 
  ChevronRight, ArrowLeft, RefreshCw, Volume2, Calendar, FileText, User, 
  Clock, Pill, Scan, Sparkles, Check, Download, CreditCard, Copy, Smartphone, ShieldAlert, Zap
} from 'lucide-react';

export default function App() {
  // Mode Switch: 'senior' | 'caregiver'
  const [activeTab, setActiveTab] = useState('senior');
  
  // Premium Status & Modal
  const [isPremium, setIsPremium] = useState(false);
  const [showCheckout, setShowCheckout] = useState(false);
  const [billingCycle, setBillingCycle] = useState('monthly'); // 'monthly' | 'yearly'
  const [copiedAccount, setCopiedAccount] = useState(false);

  // Notifications Escalation Alert
  const [alertEscalated, setAlertEscalated] = useState(false);

  // Speech State
  const [isSpeaking, setIsSpeaking] = useState(false);

  // OCR Scan Modal State
  const [showScanner, setShowScanner] = useState(false);
  const [scanStep, setScanStep] = useState('capture'); // 'capture' | 'analyzing' | 'result'
  const [scannedMedData, setScannedMedData] = useState(null);

  // PDF Report Modal
  const [showReportModal, setShowReportModal] = useState(false);

  // Initial Sample Medications State
  const [medications, setMedications] = useState([
    {
      id: '1',
      name: 'ยาลดความดัน (Amlodipine 5mg)',
      type: 'box',
      dosage: '1 เม็ด',
      time: '08:00',
      period: 'เช้า (หลังอาหาร)',
      stock: 4,
      threshold: 5,
      takenToday: true,
      takenTime: '08:05 น.',
      image: 'https://images.unsplash.com/photo-1584308666744-24d5c474f2ae?w=300&auto=format&fit=crop&q=80',
      instructions: 'ทานหลังอาหารเช้าทันที แล้วดื่มน้ำตามมากๆ'
    },
    {
      id: '2',
      name: 'ยาเบาหวาน (Metformin 500mg)',
      type: 'label',
      dosage: '1 เม็ด',
      time: '18:00',
      period: 'เย็น (หลังอาหาร)',
      stock: 14,
      threshold: 5,
      takenToday: false,
      takenTime: null,
      image: 'https://images.unsplash.com/photo-1471864190281-a93a3070b6de?w=300&auto=format&fit=crop&q=80',
      instructions: 'ทานพร้อมอาหารเย็น หรือหลังอาหารทันที'
    },
    {
      id: '3',
      name: 'วิตามินบำรุงประสาท (Vitamin B-Complex)',
      type: 'box',
      dosage: '1 เม็ด',
      time: '21:00',
      period: 'ก่อนนอน',
      stock: 20,
      threshold: 7,
      takenToday: false,
      takenTime: null,
      image: 'https://images.unsplash.com/photo-1550572017-edd951aa8f72?w=300&auto=format&fit=crop&q=80',
      instructions: 'ทานก่อนนอน 30 นาที'
    }
  ]);

  // Handle Speech Synthesis
  const speakText = (text) => {
    if ('speechSynthesis' in window) {
      window.speechSynthesis.cancel();
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = 'th-TH';
      utterance.rate = 0.9;
      utterance.onstart = () => setIsSpeaking(true);
      utterance.onend = () => setIsSpeaking(false);
      utterance.onerror = () => setIsSpeaking(false);
      window.speechSynthesis.speak(utterance);
    } else {
      alert('ระบบเล่นเสียงไม่รองรับบนเบราว์เซอร์นี้');
    }
  };

  // Toggle Medicine Taken
  const handleToggleTaken = (id) => {
    setMedications(prev => prev.map(med => {
      if (med.id === id) {
        const nextState = !med.takenToday;
        return {
          ...med,
          takenToday: nextState,
          takenTime: nextState ? new Date().toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.' : null,
          stock: nextState ? Math.max(0, med.stock - 1) : med.stock + 1
        };
      }
      return med;
    }));
  };

  // Preset OCR Scan Options
  const presets = [
    {
      title: '📦 กล่องยาลดความดัน (Amlodipine)',
      name: 'ยาลดความดัน Amlodipine 5mg',
      dosage: '1 เม็ด',
      period: 'เช้า (หลังอาหาร)',
      time: '08:00',
      stock: 30,
      image: 'https://images.unsplash.com/photo-1584308666744-24d5c474f2ae?w=300&auto=format&fit=crop&q=80',
      instructions: 'รับประทานวันละ 1 ครั้ง หลังอาหารเช้า'
    },
    {
      title: '🏷️ ซองยา รพ. ยาลดไขมัน (Simvastatin)',
      name: 'ยาลดไขมัน Simvastatin 20mg',
      dosage: '1 เม็ด',
      period: 'ก่อนนอน',
      time: '21:00',
      stock: 28,
      image: 'https://images.unsplash.com/photo-1471864190281-a93a3070b6de?w=300&auto=format&fit=crop&q=80',
      instructions: 'รับประทานวันละ 1 ครั้ง ก่อนนอน'
    },
    {
      title: '📦 กล่องยาพาราเซตามอล (Paracetamol)',
      name: 'ยาพาราเซตามอล 500mg',
      dosage: '1 เม็ด',
      period: 'ทุก 6 ชั่วโมงเมื่อมีอาการ',
      time: '12:00',
      stock: 20,
      image: 'https://images.unsplash.com/photo-1550572017-edd951aa8f72?w=300&auto=format&fit=crop&q=80',
      instructions: 'ทานทุก 4-6 ชั่วโมง เวลาปวดหรือมีไข้'
    }
  ];

  const handleSimulateScan = (preset) => {
    setScanStep('analyzing');
    setTimeout(() => {
      setScannedMedData(preset);
      setScanStep('result');
    }, 1200);
  };

  const handleSaveScannedMed = () => {
    if (!scannedMedData) return;
    const newMed = {
      id: Date.now().toString(),
      name: scannedMedData.name,
      type: 'scanned',
      dosage: scannedMedData.dosage,
      time: scannedMedData.time,
      period: scannedMedData.period,
      stock: scannedMedData.stock,
      threshold: 5,
      takenToday: false,
      takenTime: null,
      image: scannedMedData.image,
      instructions: scannedMedData.instructions
    };
    setMedications([...medications, newMed]);
    setShowScanner(false);
    setScanStep('capture');
    setScannedMedData(null);
  };

  const copyBankNumber = () => {
    navigator.clipboard.writeText('9212011435');
    setCopiedAccount(true);
    setTimeout(() => setCopiedAccount(false), 2000);
  };

  const totalMeds = medications.length;
  const takenMeds = medications.filter(m => m.takenToday).length;
  const adherenceRate = totalMeds > 0 ? Math.round((takenMeds / totalMeds) * 100) : 0;
  const lowStockMeds = medications.filter(m => m.stock <= m.threshold);

  return (
    <div className="min-h-screen bg-slate-900 text-slate-100 font-sans pb-12">
      {/* Top Main Navigation */}
      <header className="sticky top-0 z-40 bg-slate-900/90 backdrop-blur-md border-b border-slate-800">
        <div className="max-w-4xl mx-auto px-4 py-3 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="bg-gradient-to-tr from-teal-500 to-indigo-600 p-2.5 rounded-2xl shadow-lg shadow-teal-500/20">
              <Pill className="w-6 h-6 text-white" />
            </div>
            <div>
              <div className="flex items-center gap-2">
                <h1 className="font-bold text-xl tracking-tight text-white">SmartCare</h1>
                {isPremium && (
                  <span className="bg-gradient-to-r from-amber-400 to-amber-600 text-slate-950 font-black text-[10px] px-2 py-0.5 rounded-full uppercase tracking-wider flex items-center gap-1 shadow-sm">
                    <Sparkles className="w-3 h-3 fill-slate-950" /> VIP PREMIUM
                  </span>
                )}
              </div>
              <p className="text-xs text-slate-400">ระบบช่วยเตือนกินยาและดูแลผู้สูงอายุ</p>
            </div>
          </div>

          <div className="flex items-center gap-2">
            {!isPremium && (
              <button
                onClick={() => setShowCheckout(true)}
                className="hidden sm:flex items-center gap-1.5 px-3 py-1.5 rounded-xl bg-amber-500/10 border border-amber-500/30 text-amber-400 font-medium text-xs hover:bg-amber-500/20 transition-all"
              >
                <Zap className="w-3.5 h-3.5 fill-amber-400" />
                สมัคร VIP 25฿/เดือน
              </button>
            )}

            {/* Mode Switcher */}
            <div className="bg-slate-800/90 p-1 rounded-2xl border border-slate-700/60 flex items-center">
              <button
                onClick={() => setActiveTab('senior')}
                className={`flex items-center gap-2 px-3.5 py-1.5 rounded-xl font-medium text-xs transition-all ${
                  activeTab === 'senior'
                    ? 'bg-gradient-to-r from-emerald-500 to-teal-600 text-white shadow-md shadow-emerald-900/40'
                    : 'text-slate-400 hover:text-white'
                }`}
              >
                <Heart className="w-4 h-4 text-emerald-300" />
                <span>โหมดผู้สูงอายุ</span>
              </button>
              <button
                onClick={() => setActiveTab('caregiver')}
                className={`flex items-center gap-2 px-3.5 py-1.5 rounded-xl font-medium text-xs transition-all ${
                  activeTab === 'caregiver'
                    ? 'bg-gradient-to-r from-indigo-500 to-purple-600 text-white shadow-md shadow-indigo-900/40'
                    : 'text-slate-400 hover:text-white'
                }`}
              >
                <User className="w-4 h-4 text-indigo-300" />
                <span>โหมดผู้ดูแล</span>
              </button>
            </div>
          </div>
        </div>
      </header>

      {/* Main Container */}
      <main className="max-w-4xl mx-auto px-4 pt-6">
        
        {/* Banner Promotion for Non-Premium */}
        {!isPremium && (
          <div className="mb-6 rounded-3xl bg-gradient-to-r from-amber-500/15 via-purple-500/10 to-indigo-500/15 border border-amber-500/30 p-4 sm:p-5 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 relative overflow-hidden">
            <div className="flex items-center gap-3">
              <div className="p-3 bg-amber-500/20 border border-amber-500/40 rounded-2xl text-amber-400 shrink-0">
                <Sparkles className="w-6 h-6" />
              </div>
              <div>
                <h3 className="font-bold text-slate-100 text-base sm:text-lg flex items-center gap-2">
                  อัปเกรด SmartCare VIP เพียง 25 บาท / เดือน
                </h3>
                <p className="text-xs sm:text-sm text-slate-300">
                  รับระบบแจ้งเตือนเข้า LINE/SMS ลูกหลานทันทีเมื่อลืมกินยา, สแกนกล่องยาไม่จำกัด และเสียงเตือนเสียงลูกหลาน
                </p>
              </div>
            </div>
            <button
              onClick={() => setShowCheckout(true)}
              className="w-full sm:w-auto shrink-0 bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-slate-950 font-bold px-5 py-2.5 rounded-xl text-sm shadow-lg shadow-amber-500/20 transition-all flex items-center justify-center gap-2"
            >
              <CreditCard className="w-4 h-4" /> สมัครสมาชิก VIP
            </button>
          </div>
        )}

        {/* ================================================================================= */}
        {/* SENIOR MODE INTERFACE                                                            */}
        {/* ================================================================================= */}
        {activeTab === 'senior' && (
          <div className="space-y-6">
            
            {/* Senior Header & Quick Actions */}
            <div className="bg-slate-800/80 rounded-3xl border border-slate-700/80 p-5 sm:p-6 shadow-xl relative overflow-hidden">
              <div className="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4">
                <div>
                  <div className="inline-flex items-center gap-2 bg-emerald-500/10 border border-emerald-500/30 text-emerald-400 text-xs px-3 py-1 rounded-full font-medium mb-2">
                    <Clock className="w-3.5 h-3.5" /> วันนี้: {new Date().toLocaleDateString('th-TH', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}
                  </div>
                  <h2 className="text-2xl sm:text-3xl font-extrabold text-white">ตารางเตือนกินยาของคุณยาย</h2>
                  <p className="text-sm text-slate-300 mt-1">กดปุ่มสีเขียวเมื่อกินยาแล้ว หรือฟังเสียงเตือนได้ที่นี่</p>
                </div>

                <div className="flex flex-wrap items-center gap-3 w-full sm:w-auto">
                  <button
                    onClick={() => speakText(`วันนี้ มีรายการยาต้องทานทั้งหมด ${totalMeds} รายการ ทานไปแล้ว ${takenMeds} รายการค่ะ`)}
                    className="flex-1 sm:flex-none flex items-center justify-center gap-2 bg-indigo-600 hover:bg-indigo-500 text-white font-bold px-4 py-3 rounded-2xl shadow-lg shadow-indigo-600/30 transition-all text-sm"
                  >
                    <Volume2 className={`w-5 h-5 ${isSpeaking ? 'animate-bounce text-amber-300' : ''}`} />
                    <span>ฟังเสียงสรุปตารางยา</span>
                  </button>

                  <button
                    onClick={() => {
                      setScanStep('capture');
                      setShowScanner(true);
                    }}
                    className="flex-1 sm:flex-none flex items-center justify-center gap-2 bg-gradient-to-r from-teal-500 to-emerald-600 hover:from-teal-400 hover:to-emerald-500 text-white font-bold px-4 py-3 rounded-2xl shadow-lg shadow-teal-500/30 transition-all text-sm"
                  >
                    <Camera className="w-5 h-5" />
                    <span>📸 สแกนกล่อง/ฉลากยา</span>
                  </button>
                </div>
              </div>
            </div>

            {/* Senior Medication List (Extra Large Cards) */}
            <div className="space-y-4">
              {medications.map(med => (
                <div 
                  key={med.id}
                  className={`rounded-3xl border transition-all p-5 sm:p-6 shadow-xl ${
                    med.takenToday
                      ? 'bg-slate-800/40 border-slate-700/60 opacity-80'
                      : 'bg-gradient-to-br from-slate-800 to-slate-800/90 border-slate-700 shadow-teal-900/10'
                  }`}
                >
                  <div className="flex flex-col md:flex-row items-start md:items-center justify-between gap-6">
                    
                    {/* Pill Info & Image */}
                    <div className="flex items-start gap-4 sm:gap-5">
                      <img 
                        src={med.image} 
                        alt={med.name} 
                        className="w-20 h-20 sm:w-24 sm:h-24 rounded-2xl object-cover border-2 border-slate-600/80 shadow-md shrink-0 bg-slate-700"
                      />
                      <div>
                        <div className="flex items-center gap-2 mb-1">
                          <span className="bg-teal-500/20 text-teal-300 text-xs font-bold px-2.5 py-0.5 rounded-lg border border-teal-500/30">
                            🕒 {med.time} น. ({med.period})
                          </span>
                          {med.takenToday && (
                            <span className="bg-emerald-500/20 text-emerald-400 text-xs font-bold px-2 py-0.5 rounded-lg flex items-center gap-1">
                              <Check className="w-3 h-3" /> กินแล้วเมื่อ {med.takenTime}
                            </span>
                          )}
                        </div>

                        <h3 className="text-xl sm:text-2xl font-bold text-white mb-1">{med.name}</h3>
                        <p className="text-base text-amber-300 font-semibold mb-2">ทานครั้งละ: {med.dosage}</p>
                        <p className="text-xs sm:text-sm text-slate-300 bg-slate-900/60 p-2.5 rounded-xl border border-slate-700/50">
                          💡 {med.instructions}
                        </p>
                      </div>
                    </div>

                    {/* Big Action Buttons */}
                    <div className="w-full md:w-auto flex flex-col sm:flex-row md:flex-col gap-3 shrink-0">
                      <button
                        onClick={() => handleToggleTaken(med.id)}
                        className={`w-full sm:flex-1 md:w-48 py-3.5 px-4 rounded-2xl font-bold text-base sm:text-lg flex items-center justify-center gap-3 transition-all shadow-lg ${
                          med.takenToday
                            ? 'bg-slate-700 text-slate-300 hover:bg-slate-600 border border-slate-600'
                            : 'bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-400 hover:to-teal-500 text-white shadow-emerald-600/30 active:scale-95'
                        }`}
                      >
                        <CheckCircle2 className="w-6 h-6" />
                        <span>{med.takenToday ? 'ยกเลิก (กินแล้ว)' : '✅ กินยาแล้ว'}</span>
                      </button>

                      <button
                        onClick={() => speakText(`ยากินเวลานี้คือ ${med.name} ทานครั้งละ ${med.dosage} ${med.instructions}`)}
                        className="w-full sm:flex-1 md:w-48 py-2.5 px-4 rounded-2xl font-medium text-xs sm:text-sm bg-slate-700/60 hover:bg-slate-700 text-slate-200 border border-slate-600/60 flex items-center justify-center gap-2"
                      >
                        <Volume2 className="w-4 h-4 text-teal-400" />
                        <span>ฟังวิธีทานยานี้</span>
                      </button>
                    </div>

                  </div>
                </div>
              ))}
            </div>

          </div>
        )}


        {/* ================================================================================= */}
        {/* CAREGIVER MODE INTERFACE                                                         */}
        {/* ================================================================================= */}
        {activeTab === 'caregiver' && (
          <div className="space-y-6">
            
            {/* Caregiver Dashboard Header */}
            <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
              
              {/* Card 1: Adherence Score */}
              <div className="bg-slate-800/80 rounded-3xl border border-slate-700 p-5 shadow-lg">
                <div className="flex items-center justify-between mb-3">
                  <span className="text-xs text-slate-400 font-medium">วินัยการทานยาวันนี้</span>
                  <Heart className="w-5 h-5 text-rose-400" />
                </div>
                <div className="flex items-baseline gap-2 mb-2">
                  <span className="text-3xl font-extrabold text-white">{adherenceRate}%</span>
                  <span className="text-xs text-slate-400">({takenMeds}/{totalMeds} รายการ)</span>
                </div>
                <div className="w-full bg-slate-700 h-2.5 rounded-full overflow-hidden">
                  <div 
                    className="bg-gradient-to-r from-teal-400 to-emerald-500 h-full rounded-full transition-all duration-500"
                    style={{ width: `${adherenceRate}%` }}
                  ></div>
                </div>
              </div>

              {/* Card 2: Inventory Status */}
              <div className="bg-slate-800/80 rounded-3xl border border-slate-700 p-5 shadow-lg">
                <div className="flex items-center justify-between mb-3">
                  <span className="text-xs text-slate-400 font-medium">เตือนสต็อกยาใกล้หมด</span>
                  <AlertTriangle className="w-5 h-5 text-amber-400" />
                </div>
                <div className="flex items-baseline gap-2 mb-1">
                  <span className="text-3xl font-extrabold text-amber-400">{lowStockMeds.length}</span>
                  <span className="text-xs text-slate-400">รายการต้องซื้อเพิ่ม</span>
                </div>
                <p className="text-xs text-slate-400">
                  {lowStockMeds.length > 0 ? `แนะนำให้เติมยา: ${lowStockMeds.map(m=>m.name.split(' ')[0]).join(', ')}` : 'สต็อกยาทุกรายการเพียงพอ'}
                </p>
              </div>

              {/* Card 3: Quick Export & Escalation Test */}
              <div className="bg-slate-800/80 rounded-3xl border border-slate-700 p-5 shadow-lg flex flex-col justify-between">
                <div>
                  <div className="text-xs text-slate-400 font-medium mb-1">รายงาน & การแจ้งเตือนด่วน</div>
                  <p className="text-xs text-slate-300">ออกรายงานส่งคุณหมอ หรือทดลองส่ง LINE เตือน</p>
                </div>
                <div className="flex items-center gap-2 mt-3">
                  <button 
                    onClick={() => setShowReportModal(true)}
                    className="flex-1 bg-indigo-600 hover:bg-indigo-500 text-white font-bold py-2 px-3 rounded-xl text-xs flex items-center justify-center gap-1.5 transition-all"
                  >
                    <FileText className="w-4 h-4" /> สรุป PDF
                  </button>
                  <button 
                    onClick={() => {
                      setAlertEscalated(true);
                      setTimeout(() => setAlertEscalated(false), 5000);
                    }}
                    className="flex-1 bg-amber-500/20 border border-amber-500/40 text-amber-300 hover:bg-amber-500/30 font-bold py-2 px-3 rounded-xl text-xs flex items-center justify-center gap-1.5 transition-all"
                  >
                    <Bell className="w-4 h-4" /> ลองกดเตือนด่วน
                  </button>
                </div>
              </div>

            </div>

            {/* Alert Banner Escalation Test Triggered */}
            {alertEscalated && (
              <div className="bg-gradient-to-r from-rose-500 to-amber-600 p-4 rounded-2xl shadow-xl text-white flex items-center justify-between animate-pulse">
                <div className="flex items-center gap-3">
                  <ShieldAlert className="w-6 h-6 shrink-0" />
                  <div>
                    <h4 className="font-bold text-sm">📲 ส่งการแจ้งเตือนด่วนไปยัง LINE ผู้ดูแลแล้ว!</h4>
                    <p className="text-xs text-rose-100">ผู้สูงอายุยังไม่ได้กดกินยาเกิน 15 นาที ระบบส่ง Push Notification แล้ว</p>
                  </div>
                </div>
              </div>
            )}

            {/* Quick Access Import Scanner Button */}
            <div className="bg-gradient-to-r from-teal-900/40 via-slate-800 to-indigo-900/40 rounded-3xl border border-teal-500/30 p-5 shadow-xl flex flex-col sm:flex-row items-center justify-between gap-4">
              <div className="flex items-center gap-4">
                <div className="p-3.5 bg-teal-500/20 border border-teal-500/40 rounded-2xl text-teal-300">
                  <Camera className="w-7 h-7" />
                </div>
                <div>
                  <h3 className="text-lg font-bold text-white">เพิ่มรายการยาใหม่ด้วยการสแกนกล่องยา/ฉลากยา</h3>
                  <p className="text-xs text-slate-300">ถ่ายรูปกล่องยาหรือซองยา รพ. AI จะอ่านมื้อยา เวลา และจำนวนให้อัตโนมัติ</p>
                </div>
              </div>
              <button
                onClick={() => {
                  setScanStep('capture');
                  setShowScanner(true);
                }}
                className="w-full sm:w-auto bg-gradient-to-r from-teal-500 to-emerald-600 hover:from-teal-400 hover:to-emerald-500 text-white font-bold px-5 py-3 rounded-2xl text-sm shadow-lg shadow-teal-500/20 transition-all flex items-center justify-center gap-2 shrink-0"
              >
                <Scan className="w-5 h-5" /> สแกนฉลาก/กล่องยานำเข้า
              </button>
            </div>

            {/* Caregiver Medicine Inventory & Management Table */}
            <div className="bg-slate-800/80 rounded-3xl border border-slate-700 overflow-hidden shadow-xl">
              <div className="p-5 border-b border-slate-700/80 flex items-center justify-between">
                <div>
                  <h3 className="font-bold text-lg text-white">รายการยาและสต็อกคงเหลือในคลัง</h3>
                  <p className="text-xs text-slate-400">จัดการข้อมูลยา ปรับสต็อก หรือตั้งค่าการเตือน</p>
                </div>
                <span className="text-xs bg-slate-700 text-slate-300 px-3 py-1 rounded-full border border-slate-600">
                  ทั้งหมด {totalMeds} รายการ
                </span>
              </div>

              <div className="divide-y divide-slate-700/60">
                {medications.map(med => (
                  <div key={med.id} className="p-4 sm:p-5 flex flex-col sm:flex-row sm:items-center justify-between gap-4 hover:bg-slate-700/30 transition-all">
                    <div className="flex items-center gap-4">
                      <img src={med.image} alt={med.name} className="w-14 h-14 rounded-xl object-cover bg-slate-700 border border-slate-600 shrink-0" />
                      <div>
                        <div className="flex items-center gap-2 mb-1">
                          <h4 className="font-bold text-white text-base">{med.name}</h4>
                          {med.stock <= med.threshold && (
                            <span className="bg-amber-500/20 border border-amber-500/40 text-amber-300 text-[10px] font-bold px-2 py-0.5 rounded-full">
                              ⚠️ ยาใกล้หมด
                            </span>
                          )}
                        </div>
                        <p className="text-xs text-slate-300">
                          ทานครั้งละ: <span className="text-amber-300 font-medium">{med.dosage}</span> | เวลา: <span className="text-teal-300 font-medium">{med.time} น. ({med.period})</span>
                        </p>
                      </div>
                    </div>

                    <div className="flex items-center justify-between sm:justify-end gap-6 shrink-0">
                      {/* Stock Adjuster */}
                      <div className="text-right">
                        <div className="text-xs text-slate-400 mb-0.5">สต็อกคงเหลือ</div>
                        <div className="flex items-center gap-2">
                          <span className={`text-base font-extrabold ${med.stock <= med.threshold ? 'text-amber-400' : 'text-emerald-400'}`}>
                            {med.stock} เม็ด
                          </span>
                        </div>
                      </div>

                      <button
                        onClick={() => setMedications(medications.filter(m => m.id !== med.id))}
                        className="p-2 text-slate-400 hover:text-rose-400 hover:bg-rose-500/10 rounded-xl transition-all"
                        title="ลบรายการยา"
                      >
                        <Trash2 className="w-5 h-5" />
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            </div>

          </div>
        )}

      </main>


      {/* ================================================================================= */}
      {/* MODAL 1: AI OCR SCANNER (กล่องยา & ฉลากยา)                                        */}
      {/* ================================================================================= */}
      {showScanner && (
        <div className="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4">
          <div className="bg-slate-900 border border-slate-700 rounded-3xl max-w-lg w-full p-6 shadow-2xl relative overflow-hidden">
            
            <div className="flex items-center justify-between mb-4">
              <div className="flex items-center gap-2">
                <Camera className="w-5 h-5 text-teal-400" />
                <h3 className="font-bold text-lg text-white">สแกนอ่านกล่องยา / ซองยาด้วย AI</h3>
              </div>
              <button 
                onClick={() => {
                  setShowScanner(false);
                  setScanStep('capture');
                }}
                className="text-slate-400 hover:text-white p-1 rounded-lg"
              >
                ✕
              </button>
            </div>

            {/* SCAN STEP 1: CAPTURE / PRESETS */}
            {scanStep === 'capture' && (
              <div className="space-y-4">
                {/* Viewfinder Simulation */}
                <div className="relative h-48 bg-slate-950 border-2 border-dashed border-teal-500/50 rounded-2xl flex flex-col items-center justify-center p-4 text-center overflow-hidden">
                  <div className="absolute inset-4 border border-teal-400/30 rounded-xl pointer-events-none"></div>
                  <Camera className="w-10 h-10 text-teal-400 mb-2 animate-pulse" />
                  <p className="text-xs text-slate-300 font-medium">วางซองยาหรือกล่องให้อยู่ในกรอบ หรือเลือกตัวอย่างสแกนด้านล่าง</p>
                </div>

                <div className="space-y-2">
                  <p className="text-xs font-semibold text-slate-400 uppercase tracking-wider">เลือกตัวอย่างสแกนทดลอง (Preset):</p>
                  {presets.map((p, idx) => (
                    <button
                      key={idx}
                      onClick={() => handleSimulateScan(p)}
                      className="w-full p-3 rounded-xl bg-slate-800 hover:bg-slate-750 border border-slate-700 text-left flex items-center justify-between transition-all"
                    >
                      <div>
                        <div className="font-bold text-sm text-white">{p.title}</div>
                        <div className="text-xs text-slate-400">{p.period} • สต็อก {p.stock} เม็ด</div>
                      </div>
                      <ChevronRight className="w-5 h-5 text-slate-400" />
                    </button>
                  ))}
                </div>
              </div>
            )}

            {/* SCAN STEP 2: ANALYZING ANIMATION */}
            {scanStep === 'analyzing' && (
              <div className="py-12 flex flex-col items-center justify-center text-center space-y-4">
                <div className="relative">
                  <Scan className="w-16 h-16 text-teal-400 animate-spin" />
                  <Sparkles className="w-6 h-6 text-amber-400 absolute top-0 right-0 animate-ping" />
                </div>
                <div>
                  <h4 className="font-bold text-white text-lg">กำลังอ่านฉลากยาด้วย AI...</h4>
                  <p className="text-xs text-slate-400">ถอดรหัสชื่อยา มื้อเวลา และขนาดรับประทานอัตโนมัติ</p>
                </div>
              </div>
            )}

            {/* SCAN STEP 3: RESULT & CONFIRMATION */}
            {scanStep === 'result' && scannedMedData && (
              <div className="space-y-4">
                <div className="bg-teal-500/10 border border-teal-500/30 rounded-2xl p-4 flex items-center gap-3">
                  <CheckCircle2 className="w-6 h-6 text-teal-400 shrink-0" />
                  <div>
                    <h4 className="font-bold text-sm text-teal-300">สแกนสำเร็จ! ถอดข้อมูลเรียบร้อย</h4>
                    <p className="text-xs text-slate-300">ตรวจสอบข้อมูล แล้วกดบันทึกเข้าตารางเตือนทันที</p>
                  </div>
                </div>

                <div className="bg-slate-800 p-4 rounded-2xl space-y-2 border border-slate-700 text-xs">
                  <div><span className="text-slate-400">ชื่อยา:</span> <strong className="text-white text-sm">{scannedMedData.name}</strong></div>
                  <div><span className="text-slate-400">ขนาดรับประทาน:</span> <strong className="text-amber-300">{scannedMedData.dosage}</strong></div>
                  <div><span className="text-slate-400">มื้อเวลาเตือน:</span> <strong className="text-teal-300">{scannedMedData.time} น. ({scannedMedData.period})</strong></div>
                  <div><span className="text-slate-400">จำนวนยาทั้งหมด:</span> <strong className="text-emerald-300">{scannedMedData.stock} เม็ด</strong></div>
                  <div><span className="text-slate-400">คำแนะนำเพิ่มเติม:</span> <p className="text-slate-300 mt-0.5">{scannedMedData.instructions}</p></div>
                </div>

                <button
                  onClick={handleSaveScannedMed}
                  className="w-full bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-400 hover:to-teal-500 text-white font-bold py-3 rounded-xl shadow-lg shadow-emerald-600/30 text-sm flex items-center justify-center gap-2"
                >
                  <Check className="w-5 h-5" /> บันทึกเข้าตารางเตือนทันที
                </button>
              </div>
            )}

          </div>
        </div>
      )}


      {/* ================================================================================= */}
      {/* MODAL 2: CHECKOUT PREMIUM & PAYMENT (SCB 9212011435)                              */}
      {/* ================================================================================= */}
      {showCheckout && (
        <div className="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4">
          <div className="bg-slate-900 border border-slate-700 rounded-3xl max-w-md w-full p-6 shadow-2xl relative overflow-hidden">
            
            <button 
              onClick={() => setShowCheckout(false)}
              className="absolute top-4 right-4 text-slate-400 hover:text-white p-1 rounded-lg"
            >
              ✕
            </button>

            <div className="text-center mb-5">
              <div className="inline-flex p-3 bg-amber-500/20 border border-amber-500/40 rounded-2xl text-amber-400 mb-2">
                <Sparkles className="w-8 h-8" />
              </div>
              <h3 className="font-extrabold text-xl text-white">สมัคร SmartCare VIP Premium</h3>
              <p className="text-xs text-slate-400 mt-1">ดูแลคนที่คุณรักได้อย่างอุ่นใจ ไร้กังวลเรื่องลืมกินยา</p>
            </div>

            {/* Cycle Selector */}
            <div className="grid grid-cols-2 gap-2 bg-slate-800 p-1 rounded-2xl border border-slate-700 mb-5">
              <button
                onClick={() => setBillingCycle('monthly')}
                className={`py-2 text-xs font-bold rounded-xl transition-all ${
                  billingCycle === 'monthly' ? 'bg-amber-500 text-slate-950 shadow-md' : 'text-slate-400 hover:text-white'
                }`}
              >
                รายเดือน 25 บาท
              </button>
              <button
                onClick={() => setBillingCycle('yearly')}
                className={`py-2 text-xs font-bold rounded-xl transition-all ${
                  billingCycle === 'yearly' ? 'bg-amber-500 text-slate-950 shadow-md' : 'text-slate-400 hover:text-white'
                }`}
              >
                รายปี 250 บาท (ประหยัด 2 เดือน)
              </button>
            </div>

            {/* Bank Details Card (SCB 9212011435) */}
            <div className="bg-slate-800/90 border border-purple-500/40 p-4 rounded-2xl space-y-3 mb-5">
              <div className="flex items-center justify-between border-b border-slate-700 pb-2">
                <span className="text-xs text-slate-400 font-medium">โอนเงินชำระผ่านธนาคาร</span>
                <span className="text-xs font-bold text-purple-400 bg-purple-500/20 px-2 py-0.5 rounded-md">
                  ธนาคารไทยพาณิชย์ (SCB)
                </span>
              </div>

              <div>
                <div className="text-xs text-slate-400">เลขที่บัญชี:</div>
                <div className="flex items-center justify-between mt-1">
                  <span className="text-2xl font-black text-amber-400 tracking-wider">9212011435</span>
                  <button
                    onClick={copyBankNumber}
                    className="flex items-center gap-1 text-xs bg-slate-700 hover:bg-slate-600 text-slate-200 px-3 py-1.5 rounded-lg border border-slate-600 transition-all"
                  >
                    <Copy className="w-3.5 h-3.5" />
                    <span>{copiedAccount ? 'คัดลอกแล้ว!' : 'คัดลอก'}</span>
                  </button>
                </div>
                <div className="text-xs text-slate-300 mt-1">ชื่อบัญชี: บจก. สมาร์ทแคร์ เฮลท์เทค</div>
                <div className="text-xs text-amber-300 font-bold mt-1">
                  ยอดชำระ: {billingCycle === 'monthly' ? '25.00 บาท' : '250.00 บาท'}
                </div>
              </div>
            </div>

            {/* Simulate Confirm Payment */}
            <button
              onClick={() => {
                setIsPremium(true);
                setShowCheckout(false);
                alert('🎉 เปิดใช้งานสถานะ SmartCare VIP Premium เรียบร้อยแล้ว!');
              }}
              className="w-full bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-slate-950 font-extrabold py-3.5 rounded-2xl text-sm shadow-xl shadow-amber-500/20 transition-all flex items-center justify-center gap-2"
            >
              <CheckCircle2 className="w-5 h-5" /> ยืนยันการโอนเงิน / เปิดใช้งาน VIP
            </button>

          </div>
        </div>
      )}


      {/* ================================================================================= */}
      {/* MODAL 3: DOCTOR PDF REPORT PREVIEW                                                */}
      {/* ================================================================================= */}
      {showReportModal && (
        <div className="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4">
          <div className="bg-slate-900 border border-slate-700 rounded-3xl max-w-lg w-full p-6 shadow-2xl relative">
            <div className="flex items-center justify-between mb-4">
              <h3 className="font-bold text-lg text-white">รายงานสรุปการกินยาสำหรับคุณหมอ</h3>
              <button onClick={() => setShowReportModal(false)} className="text-slate-400 hover:text-white">✕</button>
            </div>

            <div className="bg-white text-slate-900 p-5 rounded-2xl text-xs space-y-3 font-mono shadow-inner">
              <div className="text-center border-b pb-2">
                <h4 className="font-bold text-base">รายงานสรุปวินัยการทานยา (SmartCare Adherence Report)</h4>
                <p className="text-slate-500">ผู้ป่วย: คุณยายสมศรี | ประจำวันที่: {new Date().toLocaleDateString('th-TH')}</p>
              </div>

              <div className="flex justify-between border-b pb-2">
                <span>อัตราการตรงเวลา (Adherence Score):</span>
                <strong className="text-emerald-700">{adherenceRate}%</strong>
              </div>

              <div>
                <strong className="block mb-1">รายละเอียดรายยามื้อวันนี้:</strong>
                {medications.map(m => (
                  <div key={m.id} className="flex justify-between py-1 border-b border-slate-100">
                    <span>{m.name} ({m.time} น.)</span>
                    <span>{m.takenToday ? `✅ กินแล้ว (${m.takenTime})` : '❌ ข้าม/ยังไม่กิน'}</span>
                  </div>
                ))}
              </div>
            </div>

            <div className="flex items-center gap-3 mt-5">
              <button 
                onClick={() => {
                  window.print();
                  setShowReportModal(false);
                }}
                className="flex-1 bg-indigo-600 hover:bg-indigo-500 text-white font-bold py-3 rounded-xl text-xs flex items-center justify-center gap-2"
              >
                <Download className="w-4 h-4" /> พิมพ์เอกสาร / บันทึกเป็น PDF
              </button>
            </div>
          </div>
        </div>
      )}

    </div>
  );
}
