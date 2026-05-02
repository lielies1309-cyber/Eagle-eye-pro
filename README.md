import React, { useState, useEffect, useMemo } from 'react';
import { 
  ChevronLeft, 
  ChevronRight, 
  Trophy, 
  LayoutGrid, 
  Plus, 
  Minus, 
  History,
  Settings,
  Circle,
  BarChart3,
  CheckCircle2
} from 'lucide-react';

const COURSE_DATA = [
  { id: 1, par: 4, index: 7, dist: 380 },
  { id: 2, par: 5, index: 15, dist: 510 },
  { id: 3, par: 3, index: 9, dist: 165 },
  { id: 4, par: 4, index: 1, dist: 420 },
  { id: 5, par: 4, index: 13, dist: 355 },
  { id: 6, par: 5, index: 3, dist: 545 },
  { id: 7, par: 3, index: 17, dist: 140 },
  { id: 8, par: 4, index: 5, dist: 395 },
  { id: 9, par: 4, index: 11, dist: 370 },
  { id: 10, par: 4, index: 8, dist: 385 },
  { id: 11, par: 3, index: 16, dist: 155 },
  { id: 12, par: 5, index: 2, dist: 530 },
  { id: 13, par: 4, index: 10, dist: 405 },
  { id: 14, par: 4, index: 4, dist: 415 },
  { id: 15, par: 3, index: 18, dist: 135 },
  { id: 16, par: 4, index: 12, dist: 365 },
  { id: 17, par: 4, index: 6, dist: 400 },
  { id: 18, par: 5, index: 14, dist: 520 },
];

const App = () => {
  const [currentHoleIndex, setCurrentHoleIndex] = useState(0);
  const [scores, setScores] = useState(Array(18).fill(0));
  const [view, setView] = useState('input'); // 'input' | 'card' | 'stats'
  const [isRoundActive, setIsRoundActive] = useState(true);

  // Derived State
  const currentHole = COURSE_DATA[currentHoleIndex];
  
  const totalPar = useMemo(() => COURSE_DATA.reduce((acc, h) => acc + h.par, 0), []);
  
  const totalScore = useMemo(() => scores.reduce((acc, s) => acc + (s || 0), 0), [scores]);
  
  const holesPlayed = useMemo(() => scores.filter(s => s > 0).length, [scores]);
  
  const currentOverUnder = useMemo(() => {
    let diff = 0;
    scores.forEach((s, i) => {
      if (s > 0) diff += (s - COURSE_DATA[i].par);
    });
    return diff;
  }, [scores]);

  const stats = useMemo(() => {
    const counts = { eagles: 0, birdies: 0, pars: 0, bogeys: 0, doubles: 0, others: 0 };
    scores.forEach((s, i) => {
      if (s === 0) return;
      const diff = s - COURSE_DATA[i].par;
      if (diff <= -2) counts.eagles++;
      else if (diff === -1) counts.birdies++;
      else if (diff === 0) counts.pars++;
      else if (diff === 1) counts.bogeys++;
      else if (diff === 2) counts.doubles++;
      else counts.others++;
    });
    return counts;
  }, [scores]);

  // Handlers
  const handleScoreChange = (mod) => {
    const newScores = [...scores];
    const currentScore = newScores[currentHoleIndex] || currentHole.par;
    newScores[currentHoleIndex] = Math.max(1, currentScore + mod);
    setScores(newScores);
  };

  const setExactScore = (val) => {
    const newScores = [...scores];
    newScores[currentHoleIndex] = val;
    setScores(newScores);
  };

  const nextHole = () => {
    if (currentHoleIndex < 17) setCurrentHoleIndex(prev => prev + 1);
  };

  const prevHole = () => {
    if (currentHoleIndex > 0) setCurrentHoleIndex(prev => prev - 1);
  };

  const resetRound = () => {
    if (confirm("Reset current round data?")) {
      setScores(Array(18).fill(0));
      setCurrentHoleIndex(0);
      setView('input');
    }
  };

  // UI Components
  const ScoreBadge = ({ score, par }) => {
    if (score === 0) return <span className="text-slate-300">-</span>;
    const diff = score - par;
    let bgColor = "bg-slate-100 text-slate-700";
    if (diff <= -2) bgColor = "bg-yellow-400 text-yellow-900 font-bold shadow-sm"; // Eagle
    if (diff === -1) bgColor = "bg-red-500 text-white font-bold shadow-sm"; // Birdie
    if (diff === 0) bgColor = "bg-emerald-600 text-white font-bold shadow-sm"; // Par
    if (diff === 1) bgColor = "bg-blue-600 text-white"; // Bogey
    if (diff >= 2) bgColor = "bg-slate-800 text-white"; // Double+

    return (
      <div className={`w-8 h-8 rounded-full flex items-center justify-center text-sm ${bgColor}`}>
        {score}
      </div>
    );
  };

  return (
    <div className="flex flex-col h-screen bg-slate-50 text-slate-900 overflow-hidden font-sans">
      {/* Top Header */}
      <header className="bg-emerald-800 text-white px-6 py-4 flex justify-between items-center shadow-lg">
        <div>
          <h1 className="text-xl font-bold flex items-center gap-2">
            <Trophy size={20} className="text-emerald-300" />
            Eagle Eye Pro
          </h1>
          <p className="text-emerald-200 text-xs font-medium uppercase tracking-wider">Live Round</p>
        </div>
        <div className="text-right">
          <p className="text-xs text-emerald-300 uppercase font-bold tracking-tighter">Total</p>
          <p className="text-2xl font-black leading-none">
            {currentOverUnder > 0 ? `+${currentOverUnder}` : currentOverUnder === 0 ? 'E' : currentOverUnder}
          </p>
        </div>
      </header>

      {/* Main Content Area */}
      <main className="flex-1 overflow-y-auto p-4 pb-24">
        
        {view === 'input' && (
          <div className="animate-in fade-in slide-in-from-bottom-4 duration-500">
            {/* Hole Selection Scroll */}
            <div className="flex gap-2 overflow-x-auto pb-4 no-scrollbar">
              {COURSE_DATA.map((h, i) => (
                <button
                  key={h.id}
                  onClick={() => setCurrentHoleIndex(i)}
                  className={`flex-shrink-0 w-12 h-12 rounded-xl flex flex-col items-center justify-center transition-all ${
                    currentHoleIndex === i 
                      ? 'bg-emerald-600 text-white shadow-md scale-110' 
                      : (scores[i] > 0 ? 'bg-emerald-100 text-emerald-800' : 'bg-white text-slate-400 border border-slate-200')
                  }`}
                >
                  <span className="text-[10px] uppercase font-bold">{h.id}</span>
                  <span className="text-sm font-bold">{scores[i] || '-'}</span>
                </button>
              ))}
            </div>

            {/* Current Hole Detail Card */}
            <div className="bg-white rounded-3xl shadow-xl p-8 border border-slate-100 text-center relative overflow-hidden">
              <div className="absolute top-0 right-0 p-4 opacity-5">
                <Circle size={150} />
              </div>
              
              <div className="flex justify-between items-start mb-6">
                <div className="text-left">
                  <h2 className="text-5xl font-black text-slate-800">Hole {currentHole.id}</h2>
                  <div className="flex gap-4 mt-2">
                    <span className="bg-slate-100 px-3 py-1 rounded-full text-xs font-bold text-slate-500 uppercase">Par {currentHole.par}</span>
                    <span className="bg-slate-100 px-3 py-1 rounded-full text-xs font-bold text-slate-500 uppercase">Hcp {currentHole.index}</span>
                    <span className="bg-slate-100 px-3 py-1 rounded-full text-xs font-bold text-slate-500 uppercase">{currentHole.dist}Y</span>
                  </div>
                </div>
                <div className="text-right">
                   <div className={`text-sm font-bold px-3 py-1 rounded-full inline-block ${
                     (scores[currentHoleIndex] - currentHole.par) < 0 ? 'bg-red-100 text-red-600' : 
                     (scores[currentHoleIndex] - currentHole.par) === 0 ? 'bg-emerald-100 text-emerald-600' : 
                     'bg-slate-100 text-slate-600'
                   }`}>
                     {scores[currentHoleIndex] > 0 ? (
                       scores[currentHoleIndex] - currentHole.par === 0 ? 'EVEN' : 
                       scores[currentHoleIndex] - currentHole.par > 0 ? `+${scores[currentHoleIndex] - currentHole.par}` : 
                       scores[currentHoleIndex] - currentHole.par
                     ) : 'READY'}
                   </div>
                </div>
              </div>

              {/* Big Score Input */}
              <div className="flex items-center justify-between mb-8 select-none">
                <button 
                  onClick={() => handleScoreChange(-1)}
                  className="w-16 h-16 rounded-full bg-slate-100 flex items-center justify-center text-slate-600 active:bg-emerald-600 active:text-white transition-colors"
                >
                  <Minus size={32} />
                </button>

                <div className="flex flex-col items-center">
                  <div className="text-8xl font-black tracking-tighter text-emerald-900 leading-none">
                    {scores[currentHoleIndex] || currentHole.par}
                  </div>
                  <p className="text-slate-400 font-bold uppercase text-xs mt-4 tracking-widest">Strokes</p>
                </div>

                <button 
                  onClick={() => handleScoreChange(1)}
                  className="w-16 h-16 rounded-full bg-slate-100 flex items-center justify-center text-slate-600 active:bg-emerald-600 active:text-white transition-colors"
                >
                  <Plus size={32} />
                </button>
              </div>

              {/* Quick Select Grid */}
              <div className="grid grid-cols-4 gap-2 mb-8">
                {[currentHole.par - 1, currentHole.par, currentHole.par + 1, currentHole.par + 2].map(v => (
                  <button
                    key={v}
                    onClick={() => setExactScore(v)}
                    className={`py-3 rounded-xl font-bold transition-all ${
                      scores[currentHoleIndex] === v 
                        ? 'bg-emerald-600 text-white shadow-inner' 
                        : 'bg-slate-50 text-slate-400 border border-slate-200'
                    }`}
                  >
                    {v === currentHole.par - 1 ? 'Birdie' : v === currentHole.par ? 'Par' : v}
                  </button>
                ))}
              </div>

              {/* Navigation */}
              <div className="flex gap-4">
                <button 
                  onClick={prevHole}
                  disabled={currentHoleIndex === 0}
                  className="flex-1 py-4 bg-slate-100 text-slate-600 rounded-2xl font-bold flex items-center justify-center gap-2 disabled:opacity-30"
                >
                  <ChevronLeft size={20} /> Back
                </button>
                <button 
                  onClick={nextHole}
                  disabled={currentHoleIndex === 17}
                  className="flex-1 py-4 bg-emerald-600 text-white rounded-2xl font-bold shadow-lg shadow-emerald-200 flex items-center justify-center gap-2 disabled:opacity-30"
                >
                  Next Hole <ChevronRight size={20} />
                </button>
              </div>
            </div>
            
            {/* Round Summary Mini Widget */}
            <div className="mt-6 flex justify-around bg-emerald-50 rounded-2xl p-4 border border-emerald-100">
               <div className="text-center">
                 <p className="text-[10px] text-emerald-600 font-bold uppercase tracking-widest">Played</p>
                 <p className="text-xl font-black text-emerald-900">{holesPlayed}/18</p>
               </div>
               <div className="text-center border-x border-emerald-200 px-8">
                 <p className="text-[10px] text-emerald-600 font-bold uppercase tracking-widest">Score</p>
                 <p className="text-xl font-black text-emerald-900">{totalScore || '-'}</p>
               </div>
               <div className="text-center">
                 <p className="text-[10px] text-emerald-600 font-bold uppercase tracking-widest">Avg Par</p>
                 <p className="text-xl font-black text-emerald-900">{holesPlayed ? (totalScore / holesPlayed).toFixed(1) : '-'}</p>
               </div>
            </div>
          </div>
        )}

        {view === 'card' && (
          <div className="animate-in fade-in zoom-in-95 duration-300">
            <h2 className="text-2xl font-black text-slate-800 mb-6">Official Scorecard</h2>
            <div className="bg-white rounded-3xl shadow-xl overflow-hidden border border-slate-200">
              <table className="w-full text-left border-collapse">
                <thead className="bg-slate-50">
                  <tr>
                    <th className="p-3 text-[10px] font-black uppercase text-slate-400">Hole</th>
                    <th className="p-3 text-[10px] font-black uppercase text-slate-400 text-center">Par</th>
                    <th className="p-3 text-[10px] font-black uppercase text-slate-400 text-center">Score</th>
                    <th className="p-3 text-[10px] font-black uppercase text-slate-400 text-center">To Par</th>
                  </tr>
                </thead>
                <tbody>
                  {COURSE_DATA.map((h, i) => (
                    <tr key={h.id} className={`${i % 2 === 0 ? '' : 'bg-slate-50/50'} border-t border-slate-100`}>
                      <td className="p-3 font-bold text-slate-600">Hole {h.id}</td>
                      <td className="p-3 text-center text-slate-400 font-medium">{h.par}</td>
                      <td className="p-3 flex justify-center">
                        <ScoreBadge score={scores[i]} par={h.par} />
                      </td>
                      <td className="p-3 text-center text-xs font-bold">
                        {scores[i] > 0 ? (
                          scores[i] - h.par > 0 ? `+${scores[i] - h.par}` : 
                          scores[i] - h.par === 0 ? 'E' : scores[i] - h.par
                        ) : '-'}
                      </td>
                    </tr>
                  ))}
                </tbody>
                <tfoot className="bg-emerald-900 text-white font-black">
                  <tr>
                    <td className="p-4">TOTAL</td>
                    <td className="p-4 text-center">{totalPar}</td>
                    <td className="p-4 text-center">{totalScore}</td>
                    <td className="p-4 text-center text-emerald-300">{currentOverUnder > 0 ? `+${currentOverUnder}` : currentOverUnder === 0 ? 'E' : currentOverUnder}</td>
                  </tr>
                </tfoot>
              </table>
            </div>
            
            <button 
              onClick={resetRound}
              className="mt-8 w-full py-4 bg-red-50 text-red-600 rounded-2xl font-bold border border-red-100 flex items-center justify-center gap-2"
            >
              <History size={20} /> End Round & Clear Data
            </button>
          </div>
        )}

        {view === 'stats' && (
          <div className="animate-in fade-in slide-in-from-right-4 duration-300">
            <h2 className="text-2xl font-black text-slate-800 mb-6">Round Analytics</h2>
            
            <div className="grid grid-cols-2 gap-4 mb-6">
               <div className="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                  <p className="text-xs font-bold text-slate-400 uppercase tracking-widest mb-1">Consistency</p>
                  <p className="text-3xl font-black text-emerald-600">{((stats.pars / holesPlayed) * 100 || 0).toFixed(0)}%</p>
                  <p className="text-[10px] text-slate-400">GIR Estimate</p>
               </div>
               <div className="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                  <p className="text-xs font-bold text-slate-400 uppercase tracking-widest mb-1">Pace</p>
                  <p className="text-3xl font-black text-blue-600">{((totalScore / totalPar) * 100 || 0).toFixed(0)}%</p>
                  <p className="text-[10px] text-slate-400">vs Course Par</p>
               </div>
            </div>

            <div className="bg-white rounded-3xl shadow-lg p-6 border border-slate-100">
              <h3 className="font-bold text-slate-800 mb-4 flex items-center gap-2">
                <BarChart3 size={18} className="text-emerald-600" />
                Score Distribution
              </h3>
              
              <div className="space-y-4">
                {[
                  { label: 'Eagle or Better', count: stats.eagles, color: 'bg-yellow-400' },
                  { label: 'Birdies', count: stats.birdies, color: 'bg-red-500' },
                  { label: 'Pars', count: stats.pars, color: 'bg-emerald-600' },
                  { label: 'Bogeys', count: stats.bogeys, color: 'bg-blue-600' },
                  { label: 'Double Bogeys', count: stats.doubles, color: 'bg-slate-800' },
                  { label: 'Others', count: stats.others, color: 'bg-slate-300' },
                ].map(item => (
                  <div key={item.label}>
                    <div className="flex justify-between text-xs font-bold text-slate-500 mb-1">
                      <span>{item.label}</span>
                      <span>{item.count}</span>
                    </div>
                    <div className="h-3 bg-slate-100 rounded-full overflow-hidden">
                      <div 
                        className={`h-full ${item.color} transition-all duration-1000`} 
                        style={{ width: `${holesPlayed ? (item.count / holesPlayed) * 100 : 0}%` }}
                      ></div>
                    </div>
                  </div>
                ))}
              </div>
            </div>
            
            <div className="mt-6 bg-emerald-900 rounded-3xl p-8 text-white text-center shadow-xl">
               <CheckCircle2 size={48} className="mx-auto mb-4 text-emerald-400" />
               <h3 className="text-xl font-bold mb-2">Round in Progress</h3>
               <p className="text-emerald-200 text-sm mb-6">You are currently through {holesPlayed} holes at {currentOverUnder > 0 ? `+${currentOverUnder}` : currentOverUnder === 0 ? 'Even' : currentOverUnder}. Keep up the pace!</p>
            </div>
          </div>
        )}

      </main>

      {/* Bottom Navigation */}
      <nav className="fixed bottom-0 left-0 right-0 bg-white border-t border-slate-200 px-6 py-4 flex justify-around items-center shadow-2xl safe-area-bottom">
        <button 
          onClick={() => setView('input')}
          className={`flex flex-col items-center gap-1 transition-colors ${view === 'input' ? 'text-emerald-600 scale-110' : 'text-slate-400'}`}
        >
          <Plus size={24} strokeWidth={view === 'input' ? 3 : 2} />
          <span className="text-[10px] font-bold uppercase tracking-tighter">Enter Score</span>
        </button>
        
        <button 
          onClick={() => setView('card')}
          className={`flex flex-col items-center gap-1 transition-colors ${view === 'card' ? 'text-emerald-600 scale-110' : 'text-slate-400'}`}
        >
          <LayoutGrid size={24} strokeWidth={view === 'card' ? 3 : 2} />
          <span className="text-[10px] font-bold uppercase tracking-tighter">Scorecard</span>
        </button>
        
        <button 
          onClick={() => setView('stats')}
          className={`flex flex-col items-center gap-1 transition-colors ${view === 'stats' ? 'text-emerald-600 scale-110' : 'text-slate-400'}`}
        >
          <BarChart3 size={24} strokeWidth={view === 'stats' ? 3 : 2} />
          <span className="text-[10px] font-bold uppercase tracking-tighter">Analytics</span>
        </button>
      </nav>

      <style dangerouslySetInnerHTML={{ __html: `
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        .safe-area-bottom { padding-bottom: calc(1rem + env(safe-area-inset-bottom)); }
      `}} />
    </div>
  );
};

export default App;
