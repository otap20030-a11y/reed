
import React, { useState } from 'react';
import { MEXICAN_READING_TEXTS } from './constants';
import { ReadingResult, GradeText } from './types';
import { analyzeReadingAudio } from './services/geminiService';
import AudioRecorder from './components/AudioRecorder';
import ResultsView from './components/ResultsView';

const App: React.FC = () => {
  const [studentName, setStudentName] = useState<string>('');
  const [selectedGrade, setSelectedGrade] = useState<number>(1);
  const [selectedText, setSelectedText] = useState<GradeText | null>(MEXICAN_READING_TEXTS[0]);
  const [result, setResult] = useState<ReadingResult | null>(null);
  const [isEvaluating, setIsEvaluating] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleGradeChange = (grade: number) => {
    setSelectedGrade(grade);
    const textForGrade = MEXICAN_READING_TEXTS.find(t => t.grade === grade);
    setSelectedText(textForGrade || null);
    setResult(null);
    setError(null);
  };

  const handleRecordingComplete = async (audioBase64: string, duration: number) => {
    if (!selectedText) return;

    setIsEvaluating(true);
    setError(null);
    try {
      const evaluation = await analyzeReadingAudio(
        audioBase64,
        selectedText.content,
        selectedGrade,
        duration
      );
      setResult(evaluation);
    } catch (err) {
      console.error(err);
      setError("No se pudo completar el análisis. Intenta grabar de nuevo con una voz más clara.");
    } finally {
      setIsEvaluating(false);
    }
  };

  const resetApp = () => {
    setResult(null);
    setError(null);
  };

  return (
    <div className="min-h-screen bg-slate-50 pb-12">
      {/* SISAT Style Header */}
      <header className="bg-white border-b shadow-sm sticky top-0 z-20">
        <div className="max-w-6xl mx-auto px-4 h-16 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="bg-emerald-700 text-white p-2 rounded shadow-md">
              <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253" />
              </svg>
            </div>
            <div>
              <h1 className="font-extrabold text-slate-800 text-lg leading-tight">Evaluación SISAT</h1>
              <p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest">Comprensión y Fluidez Lectora</p>
            </div>
          </div>
          
          <div className="hidden sm:flex items-center gap-2">
            <span className="text-xs font-bold text-slate-400 mr-2">GRADO:</span>
            <div className="flex bg-slate-100 p-1 rounded-lg">
              {[1, 2, 3, 4, 5, 6].map(g => (
                <button
                  key={g}
                  onClick={() => handleGradeChange(g)}
                  className={`w-9 h-9 flex items-center justify-center rounded-md text-sm font-bold transition-all ${
                    selectedGrade === g 
                    ? 'bg-white text-emerald-700 shadow-sm scale-110' 
                    : 'text-slate-400 hover:text-slate-600'
                  }`}
                >
                  {g}°
                </button>
              ))}
            </div>
          </div>
        </div>
      </header>

      {/* Main Content */}
      <main className="max-w-6xl mx-auto px-4 mt-8">
        {!result ? (
          <div className="grid grid-cols-1 lg:grid-cols-12 gap-8">
            {/* Config & Student Details */}
            <div className="lg:col-span-4 space-y-6">
              <section className="bg-white p-6 rounded-2xl border shadow-sm">
                <h3 className="text-sm font-bold text-slate-400 uppercase tracking-widest mb-4">Datos del Alumno</h3>
                <div className="space-y-4">
                  <div>
                    <label className="block text-xs font-bold text-slate-600 mb-1">NOMBRE COMPLETO:</label>
                    <input 
                      type="text" 
                      placeholder="Ej. Juan Pérez López"
                      className="w-full px-4 py-2 bg-slate-50 border rounded-xl text-slate-800 focus:outline-none focus:ring-2 focus:ring-emerald-500 transition-all"
                      value={studentName}
                      onChange={(e) => setStudentName(e.target.value)}
                    />
                  </div>
                  <div className="sm:hidden">
                    <label className="block text-xs font-bold text-slate-600 mb-2">GRADO ESCOLAR:</label>
                    <div className="flex gap-2 flex-wrap">
                      {[1, 2, 3, 4, 5, 6].map(g => (
                        <button
                          key={g}
                          onClick={() => handleGradeChange(g)}
                          className={`w-10 h-10 flex items-center justify-center rounded-xl font-bold border ${
                            selectedGrade === g ? 'bg-emerald-600 text-white' : 'bg-white text-slate-500'
                          }`}
                        >
                          {g}
                        </button>
                      ))}
                    </div>
                  </div>
                </div>
              </section>

              <section className="bg-emerald-800 p-6 rounded-2xl text-white shadow-lg overflow-hidden relative">
                <div className="relative z-10">
                  <h3 className="font-bold mb-2">Instrucciones SISAT:</h3>
                  <ul className="text-sm space-y-2 opacity-90">
                    <li className="flex gap-2"><span>1.</span> Pide al alumno que lea a su ritmo normal.</li>
                    <li className="flex gap-2"><span>2.</span> Presiona grabar y detén al terminar el texto.</li>
                    <li className="flex gap-2"><span>3.</span> Revisa los errores y el nivel de desempeño.</li>
                  </ul>
                </div>
                <div className="absolute -bottom-4 -right-4 text-emerald-700 opacity-20 transform rotate-12">
                  <svg xmlns="http://www.w3.org/2000/svg" className="h-24 w-24" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z" />
                  </svg>
                </div>
              </section>
            </div>

            {/* Reading Material & Recorder */}
            <div className="lg:col-span-8 space-y-6">
              <section className="bg-white p-8 rounded-3xl border shadow-sm">
                <div className="flex items-center justify-between mb-8">
                  <div>
                    <span className="text-emerald-600 text-xs font-black uppercase tracking-widest">Material de Grado {selectedGrade}°</span>
                    <h2 className="text-3xl font-black text-slate-800">{selectedText?.title}</h2>
                  </div>
                  <div className="text-right">
                    <span className="block text-2xl font-bold text-slate-300">#{selectedGrade}</span>
                  </div>
                </div>

                <div className="bg-slate-50 p-8 rounded-2xl border border-slate-100 mb-8">
                  <p className="text-2xl leading-relaxed text-slate-700 font-serif">
                    {selectedText?.content}
                  </p>
                </div>

                {isEvaluating ? (
                  <div className="py-12 flex flex-col items-center justify-center gap-4 bg-emerald-50 rounded-2xl border border-emerald-100 animate-pulse">
                    <div className="w-12 h-12 border-4 border-emerald-600 border-t-transparent rounded-full animate-spin"></div>
                    <div className="text-center">
                      <p className="font-bold text-emerald-800">Generando Reporte SISAT...</p>
                      <p className="text-xs text-emerald-600">Analizando fluidez, velocidad y precisión</p>
                    </div>
                  </div>
                ) : (
                  <div className="bg-white border-2 border-dashed border-slate-200 rounded-2xl p-4">
                    <AudioRecorder 
                      onRecordingComplete={handleRecordingComplete} 
                      disabled={!selectedText || isEvaluating}
                    />
                  </div>
                )}

                {error && (
                  <div className="mt-4 p-4 bg-rose-50 border border-rose-100 text-rose-600 text-sm rounded-xl flex items-center gap-3">
                    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                      <path fillRule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z" clipRule="evenodd" />
                    </svg>
                    {error}
                  </div>
                )}
              </section>
            </div>
          </div>
        ) : (
          <div className="space-y-6">
             <div className="bg-white px-8 py-4 rounded-2xl border shadow-sm flex items-center justify-between">
                <div>
                  <h2 className="text-slate-400 text-xs font-bold uppercase tracking-widest">Reporte Individual</h2>
                  <p className="text-xl font-bold text-slate-800">{studentName || 'Alumno sin nombre'}</p>
                </div>
                <div className="text-right">
                  <p className="text-xs font-bold text-slate-400">FECHA</p>
                  <p className="text-sm font-bold text-slate-600">{new Date().toLocaleDateString('es-MX')}</p>
                </div>
             </div>
             <ResultsView 
                result={result} 
                grade={selectedGrade} 
                studentName={studentName}
                onReset={resetApp} 
              />
          </div>
        )}
      </main>
    </div>
  );
};

export default App;
