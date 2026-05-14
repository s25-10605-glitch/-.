# -.
import React, { useState, useEffect } from 'react';
import { 
  Save, 
  Trash2, 
  CheckCircle2, 
  AlertCircle, 
  LayoutDashboard, 
  PencilLine, 
  BookOpen, 
  Users, 
  Lightbulb,
  FileText,
  Plus
} from 'lucide-react';

/**
 * [GitHub 업로드 및 배포 가이드]
 * 1. 빈 폴더를 만들고 'npm create vite@latest .' 명령어로 React 프로젝트를 생성하세요.
 * 2. 'npm install lucide-react' 명령어로 아이콘 라이브러리를 설치하세요.
 * 3. 이 코드를 App.jsx 파일에 붙여넣으세요.
 * 4. GitHub에 Push하면 Vercel이나 Netlify에서 자동으로 인식하여 배포됩니다.
 */

const App = () => {
  const [activeTab, setActiveTab] = useState('editor');
  const [records, setRecords] = useState([]);
  const [currentRecord, setCurrentRecord] = useState({
    id: Date.now(),
    type: 'subject_spec',
    title: '',
    content: '',
    subject: '',
    date: new Date().toISOString().split('T')[0]
  });

  const recordTypes = {
    subject_spec: { label: '교과 세특', limit: 1500, color: 'bg-blue-100 text-blue-700' },
    autonomous: { label: '자율활동', limit: 1500, color: 'bg-purple-100 text-purple-700' },
    club: { label: '동아리활동', limit: 1500, color: 'bg-green-100 text-green-700' },
    career: { label: '진로활동', limit: 2100, color: 'bg-orange-100 text-orange-700' },
    behavior: { label: '행동특성', limit: 1500, color: 'bg-pink-100 text-pink-700' }
  };

  useEffect(() => {
    const saved = localStorage.getItem('student_records');
    if (saved) {
      try {
        setRecords(JSON.parse(saved));
      } catch (e) {
        console.error("Failed to parse records", e);
      }
    }
  }, []);

  const saveRecords = (newRecords) => {
    localStorage.setItem('student_records', JSON.stringify(newRecords));
    setRecords(newRecords);
  };

  const getByteLength = (str) => {
    if (!str) return 0;
    let bytes = 0;
    for (let i = 0; i < str.length; i++) {
      const code = str.charCodeAt(i);
      bytes += (code > 128) ? 3 : 1;
    }
    return bytes;
  };

  const handleSave = () => {
    if (!currentRecord.title || !currentRecord.content) {
      alert('제목과 내용을 입력해주세요.');
      return;
    }
    const exists = records.find(r => r.id === currentRecord.id);
    let newRecords;
    if (exists) {
      newRecords = records.map(r => r.id === currentRecord.id ? currentRecord : r);
    } else {
      newRecords = [currentRecord, ...records];
    }
    saveRecords(newRecords);
    alert('저장되었습니다.');
  };

  const handleDelete = (id) => {
    if (window.confirm('정말 삭제하시겠습니까?')) {
      const newRecords = records.filter(r => r.id !== id);
      saveRecords(newRecords);
    }
  };

  const startNew = () => {
    setCurrentRecord({
      id: Date.now(),
      type: 'subject_spec',
      title: '',
      content: '',
      subject: '',
      date: new Date().toISOString().split('T')[0]
    });
    setActiveTab('editor');
  };

  const editRecord = (record) => {
    setCurrentRecord(record);
    setActiveTab('editor');
  };

  const convertToSchoolStyle = () => {
    let text = currentRecord.content;
    text = text.replace(/했습니다\./g, '함.');
    text = text.replace(/하였습니다\./g, '함.');
    text = text.replace(/입니다\./g, '임.');
    text = text.replace(/보였습니다\./g, '보임.');
    text = text.replace(/느꼈습니다\./g, '느낌.');
    setCurrentRecord({ ...currentRecord, content: text });
  };

  const byteCount = getByteLength(currentRecord.content);
  const limit = recordTypes[currentRecord.type].limit;
  const isOverLimit = byteCount > limit;

  return (
    <div className="min-h-screen bg-slate-50 font-sans text-slate-900">
      <nav className="bg-white border-b border-slate-200 sticky top-0 z-10">
        <div className="max-w-5xl mx-auto px-4 h-16 flex items-center justify-between">
          <div className="flex items-center space-x-2">
            <div className="bg-indigo-600 p-2 rounded-lg">
              <FileText className="text-white w-5 h-5" />
            </div>
            <span className="font-bold text-xl tracking-tight text-indigo-900">생기부 매니저</span>
          </div>
          <div className="flex space-x-1">
            <button 
              onClick={() => setActiveTab('dashboard')}
              className={`px-4 py-2 rounded-md flex items-center space-x-2 transition ${activeTab === 'dashboard' ? 'bg-indigo-50 text-indigo-700' : 'text-slate-600 hover:bg-slate-100'}`}
            >
              <LayoutDashboard size={18} />
              <span className="hidden sm:inline font-medium">대시보드</span>
            </button>
            <button 
              onClick={() => setActiveTab('editor')}
              className={`px-4 py-2 rounded-md flex items-center space-x-2 transition ${activeTab === 'editor' ? 'bg-indigo-50 text-indigo-700' : 'text-slate-600 hover:bg-slate-100'}`}
            >
              <PencilLine size={18} />
              <span className="hidden sm:inline font-medium">작성하기</span>
            </button>
          </div>
        </div>
      </nav>

      <main className="max-w-5xl mx-auto px-4 py-8">
        {activeTab === 'dashboard' ? (
          <div className="space-y-6 animate-in fade-in duration-500">
            <div className="flex justify-between items-center">
              <h2 className="text-2xl font-bold text-slate-800">내 기록 목록</h2>
              <button 
                onClick={startNew}
                className="bg-indigo-600 text-white px-4 py-2 rounded-lg hover:bg-indigo-700 transition flex items-center space-x-2 shadow-sm"
              >
                <Plus size={18} />
                <span>새 기록 작성</span>
              </button>
            </div>

            {records.length === 0 ? (
              <div className="bg-white border-2 border-dashed border-slate-200 rounded-xl p-12 text-center">
                <BookOpen className="mx-auto w-12 h-12 text-slate-300 mb-4" />
                <p className="text-slate-500 text-lg font-medium">아직 작성된 기록이 없습니다. 첫 생기부 초안을 작성해보세요!</p>
              </div>
            ) : (
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                {records.map(record => (
                  <div key={record.id} className="bg-white border border-slate-200 rounded-xl p-5 hover:shadow-md transition group relative">
                    <div className="flex justify-between items-start mb-3">
                      <span className={`text-xs font-bold px-2.5 py-1 rounded-full ${recordTypes[record.type].color}`}>
                        {recordTypes[record.type].label}
                      </span>
                      <div className="flex space-x-2 sm:opacity-0 group-hover:opacity-100 transition-opacity">
                        <button onClick={() => editRecord(record)} className="p-1.5 text-slate-400 hover:text-indigo-600 bg-slate-50 rounded">
                          <PencilLine size={16} />
                        </button>
                        <button onClick={() => handleDelete(record.id)} className="p-1.5 text-slate-400 hover:text-red-600 bg-slate-50 rounded">
                          <Trash2 size={16} />
                        </button>
                      </div>
                    </div>
                    <h3 className="font-bold text-lg mb-1 text-slate-800 truncate pr-16">{record.title}</h3>
                    <p className="text-slate-500 text-sm mb-4 line-clamp-2 leading-relaxed">{record.content}</p>
                    <div className="flex justify-between items-center text-xs text-slate-400 border-t border-slate-50 pt-3">
                      <span className="font-medium">{record.date} {record.subject && `| ${record.subject}`}</span>
                      <span className="bg-slate-100 px-2 py-0.5 rounded font-bold">{getByteLength(record.content)} Byte</span>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        ) : (
          <div className="grid grid-cols-1 lg:grid-cols-3 gap-8 animate-in slide-in-from-bottom-4 duration-500">
            <div className="lg:col-span-2 space-y-6">
              <div className="bg-white border border-slate-200 rounded-2xl shadow-sm overflow-hidden">
                <div className="p-6 border-b border-slate-100 bg-slate-50/50 flex justify-between items-center">
                  <h2 className="font-bold text-slate-800 flex items-center space-x-2">
                    <PencilLine size={18} className="text-indigo-600" />
                    <span>활동 내용 작성</span>
                  </h2>
                  <div className="flex space-x-2">
                    <button 
                      onClick={convertToSchoolStyle}
                      className="text-xs bg-white border border-slate-200 px-3 py-1.5 rounded-md text-slate-600 hover:bg-slate-50 flex items-center space-x-1 font-medium shadow-sm transition"
                      title="문장 끝을 '~함', '~임'으로 자동 변환합니다."
                    >
                      <Lightbulb size={14} className="text-amber-500" />
                      <span>말투 변환</span>
                    </button>
                    <button 
                      onClick={handleSave}
                      className="text-xs bg-indigo-600 px-4 py-1.5 rounded-md text-white hover:bg-indigo-700 flex items-center space-x-1 font-medium shadow-sm transition"
                    >
                      <Save size={14} />
                      <span>저장</span>
                    </button>
                  </div>
                </div>
                
                <div className="p-6 space-y-5">
                  <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <div className="space-y-1.5">
                      <label className="text-xs font-bold text-slate-500 uppercase tracking-wider">항목 분류</label>
                      <select 
                        value={currentRecord.type}
                        onChange={(e) => setCurrentRecord({...currentRecord, type: e.target.value})}
                        className="w-full bg-slate-50 border border-slate-200 rounded-lg px-3 py-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none transition"
                      >
                        {Object.entries(recordTypes).map(([key, value]) => (
                          <option key={key} value={key}>{value.label}</option>
                        ))}
                      </select>
                    </div>
                    <div className="space-y-1.5">
                      <label className="text-xs font-bold text-slate-500 uppercase tracking-wider">과목명 (선택)</label>
                      <input 
                        type="text" 
                        value={currentRecord.subject}
                        onChange={(e) => setCurrentRecord({...currentRecord, subject: e.target.value})}
                        placeholder="예: 수학 I"
                        className="w-full bg-slate-50 border border-slate-200 rounded-lg px-3 py-2.5 text-sm focus:ring-2 focus:ring-indigo-500 outline-none transition"
                      />
                    </div>
                  </div>

                  <div className="space-y-1.5">
                    <label className="text-xs font-bold text-slate-500 uppercase tracking-wider">활동 제목</label>
                    <input 
                      type="text" 
                      value={currentRecord.title}
                      onChange={(e) => setCurrentRecord({...currentRecord, title: e.target.value})}
                      placeholder="활동의 핵심 주제를 적어주세요"
                      className="w-full bg-slate-50 border border-slate-200 rounded-lg px-4 py-2.5 font-semibold focus:ring-2 focus:ring-indigo-500 outline-none transition"
                    />
                  </div>

                  <div className="space-y-1.5">
                    <div className="flex justify-between items-end mb-1">
                      <label className="text-xs font-bold text-slate-500 uppercase tracking-wider">상세 내용</label>
                      <div className={`text-xs font-bold px-2 py-0.5 rounded-full ${isOverLimit ? 'bg-red-100 text-red-600' : 'bg-slate-100 text-slate-600'}`}>
                        {byteCount} / {limit} Byte
                      </div>
                    </div>
                    <textarea 
                      value={currentRecord.content}
                      onChange={(e) => setCurrentRecord({...currentRecord, content: e.target.value})}
                      placeholder="계기 - 활동 과정 - 깨달은 점 및 변화 순으로 작성하면 좋습니다."
                      className="w-full h-[450px] bg-slate-50 border border-slate-200 rounded-lg px-4 py-3 text-base leading-relaxed focus:ring-2 focus:ring-indigo-500 outline-none resize-none transition"
                    />
                  </div>
                </div>
              </div>
            </div>

            <div className="space-y-6">
              <div className="bg-gradient-to-br from-indigo-700 to-indigo-900 text-white rounded-2xl p-6 shadow-xl shadow-indigo-100 border border-indigo-500">
                <h3 className="font-bold text-lg mb-4 flex items-center space-x-2 border-b border-indigo-500/50 pb-3">
                  <CheckCircle2 size={20} className="text-indigo-300" />
                  <span>작성 핵심 팁</span>
                </h3>
                <ul className="space-y-4 text-sm text-indigo-50">
                  <li className="flex items-start space-x-3">
                    <span className="bg-indigo-600 text-indigo-200 rounded-full w-5 h-5 flex items-center justify-center text-[10px] font-bold shrink-0 mt-0.5 border border-indigo-400">01</span>
                    <span className="leading-tight">나열식 기록보다는 구체적인 **동기**와 본인만의 **심화 활동**을 서술하세요.</span>
                  </li>
                  <li className="flex items-start space-x-3">
                    <span className="bg-indigo-600 text-indigo-200 rounded-full w-5 h-5 flex items-center justify-center text-[10px] font-bold shrink-0 mt-0.5 border border-indigo-400">02</span>
                    <span>활동 중 맞닥뜨린 **문제 상황**과 이를 **극복한 과정**이 가장 매력적입니다.</span>
                  </li>
                  <li className="flex items-start space-x-3">
                    <span className="bg-indigo-600 text-indigo-200 rounded-full w-5 h-5 flex items-center justify-center text-[10px] font-bold shrink-0 mt-0.5 border border-indigo-400">03</span>
                    <span>수업 시간에 배운 개념을 기반으로 추가적인 **독서**나 **보고서**를 연계하세요.</span>
                  </li>
                </ul>
              </div>

              <div className="bg-white border border-slate-200 rounded-2xl p-6 shadow-sm">
                <h3 className="font-bold text-slate-800 mb-4 flex items-center space-x-2">
                  <AlertCircle size={20} className="text-amber-500" />
                  <span>필수 유의 사항</span>
                </h3>
                <div className="space-y-3">
                  <div className="p-4 bg-red-50 rounded-xl border border-red-100">
                    <p className="text-xs font-bold text-red-700 mb-1">기재 금지 키워드</p>
                    <p className="text-xs text-red-600 leading-normal">
                      어학성적, 교외 수상 실적, 부모님 직업, 대학 명칭, 구체적 지역명 등은 입학 취소 사유가 될 수 있습니다.
                    </p>
                  </div>
                  <div className="p-4 bg-slate-50 rounded-xl border border-slate-100">
                    <p className="text-xs font-bold text-slate-700 mb-1">나이스(NEIS) 바이트 계산</p>
                    <p className="text-xs text-slate-600 leading-normal">
                      NEIS는 한글 1자=3Byte, 공백/영문 1자=1Byte로 계산합니다. 본 서비스는 이 기준을 따릅니다.
                    </p>
                  </div>
                </div>
              </div>

              <div className="bg-white border border-slate-200 rounded-2xl p-6 shadow-sm">
                <h3 className="font-bold text-slate-800 mb-4 flex items-center space-x-2">
                  <Users size={20} className="text-emerald-500" />
                  <span>주요 역량 키워드</span>
                </h3>
                <div className="flex flex-wrap gap-2">
                  {['학업역량', '진로역량', '공동체역량', '자기주도성', '문제해결력', '비판적 사고', '의사소통능력'].map(tag => (
                    <span key={tag} className="text-[11px] font-bold bg-slate-100 text-slate-600 px-2.5 py-1.5 rounded-lg border border-slate-200 cursor-default hover:bg-indigo-50 hover:text-indigo-600 hover:border-indigo-100 transition-colors">
                      #{tag}
                    </span>
                  ))}
                </div>
              </div>
            </div>
          </div>
        )}
      </main>
      
      <footer className="max-w-5xl mx-auto px-4 py-12 text-center">
        <div className="h-px bg-slate-200 w-full mb-6"></div>
        <p className="text-slate-400 text-xs font-medium">© 2024 생기부 매니저. 모든 데이터는 브라우저 내에 안전하게 저장됩니다.</p>
      </footer>
    </div>
  );
};

export default App;
