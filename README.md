import React, { useState, useEffect, useCallback, useMemo } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { 
    getFirestore, doc, setDoc, onSnapshot, collection, addDoc, updateDoc, deleteDoc, getDocs, query, where, writeBatch
} from 'firebase/firestore';

// Tailwind CSS ikonları için Lucide React'i simüle ediyoruz.
const XIcon = (props) => (
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
);
const CheckIcon = (props) => (
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="20 6 9 17 4 12"></polyline></svg>
);
const PlusIcon = (props) => (
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>
);
const TrashIcon = (props) => (
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
);
const BotIcon = (props) => (
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M12 8V4h-1M12 20h-1M18 10h-2V7h-4V4H8M6 10h-2V7h-4V4H0M16 10v4M8 10v4M12 16v4M4 16v4M20 16v4M12 20h1"></path></svg>
);
const SaveIcon = (props) => (
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"></path><polyline points="17 21 17 13 7 13 7 21"></polyline><polyline points="7 3 7 8 15 8"></polyline></svg>
);
const ClockIcon = (props) => (
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline></svg>
);
const ZapIcon = (props) => (
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"></polygon></svg>
);
const AlertTriangleIcon = (props) => ( 
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path><line x1="12" y1="9" x2="12" y2="13"></line><line x1="12" y1="17" x2="12.01" y2="17"></line></svg>
);
const ActivityIcon = (props) => ( // Yeni İkon
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"></polyline></svg>
);
const MessageCircleIcon = (props) => ( // Yeni İkon
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8 8.5 8.5 0 0 1-7.6 4.7 8.38 8.38 0 0 1-3.8-.9L3 21l1.9-5.7.1-.4.1-.2H2.5A2.5 2.5 0 0 1 0 12.5v-7A2.5 2.5 0 0 1 2.5 3h19A2.5 2.5 0 0 1 24 5.5v6z"></path></svg>
);
const SettingsIcon = (props) => ( // Yeni İkon
    <svg {...props} xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M12.22 2h-.44a2 2 0 0 0-2 2v.44a2 2 0 0 1-1.11 1.79l-.31.18a2 2 0 0 0-2.6 1.13l-.22.42a2 2 0 0 0 .54 2.19l.34.33a2 2 0 0 1 .44 1.34v.44a2 2 0 0 1-.44 1.34l-.34.33a2 2 0 0 0-.54 2.19l.22.42a2 2 0 0 0 2.6 1.13l.31.18a2 2 0 0 1 1.11 1.79v.44a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.44a2 2 0 0 1 1.11-1.79l.31-.18a2 2 0 0 0 2.6-1.13l.22-.42a2 2 0 0 0-.54-2.19l-.34-.33a2 2 0 0 1-.44-1.34v-.44a2 2 0 0 1 .44-1.34l.34-.33a2 2 0 0 0 .54-2.19l-.22-.42a2 2 0 0 0-2.6-1.13l-.31-.18a2 2 0 0 1-1.11-1.79V4a2 2 0 0 0-2-2z"></path><circle cx="12" cy="12" r="3"></circle></svg>
);

// --- Sabitler ve Yardımcı Fonksiyonlar ---

const API_KEY = ""; // Canvas ortamında otomatik olarak sağlanır
const GEMINI_API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent";

const STATUSES = {
    TODO: 'Yapılacak',
    IN_PROGRESS: 'Devam Ediyor',
    DONE: 'Tamamlandı',
};

const PRIORITIES = {
    HIGH: 'Yüksek',
    MEDIUM: 'Orta',
    LOW: 'Düşük',
};

// Renk Fonksiyonları

const getColumnColor = (status) => {
    switch (status) {
        case STATUSES.TODO: return 'bg-gray-200 text-gray-800 shadow-md'; // Muted Gray
        case STATUSES.IN_PROGRESS: return 'bg-blue-500 text-white shadow-md'; // Standard Blue
        case STATUSES.DONE: return 'bg-green-500 text-white shadow-md'; // Standard Green
        default: return 'bg-gray-400 text-white';
    }
};

const getStatusBadgeColor = (status) => {
    switch (status) {
        case STATUSES.TODO: return 'bg-gray-100 text-gray-700 border-gray-300';
        case STATUSES.IN_PROGRESS: return 'bg-blue-100 text-blue-700 border-blue-300';
        case STATUSES.DONE: return 'bg-green-100 text-green-700 border-green-300';
        default: return 'bg-gray-100 text-gray-700 border-gray-300';
    }
};

const getPriorityColor = (priority) => {
    switch (priority) {
        case PRIORITIES.HIGH: return 'bg-red-600 text-white border-red-800';
        case PRIORITIES.MEDIUM: return 'bg-yellow-400 text-gray-900 border-yellow-600';
        case PRIORITIES.LOW: return 'bg-green-500 text-white border-green-700';
        default: return 'bg-gray-400 text-white border-gray-600';
    }
};

// Görev kartının arka plan rengini önceliğe göre belirler
const getPriorityBgColor = (priority) => {
    switch (priority) {
        case PRIORITIES.HIGH: return 'bg-red-50 hover:bg-red-100 border-red-300'; // Yüksek öncelik
        case PRIORITIES.MEDIUM: return 'bg-orange-50 hover:bg-orange-100 border-orange-300'; // Orta öncelik
        case PRIORITIES.LOW: return 'bg-blue-50 hover:bg-blue-100 border-blue-300'; // Düşük öncelik
        default: return 'bg-white hover:bg-gray-100 border-gray-300';
    }
};

// Aciliyet Seviyesini Belirler
const getUrgencyLevel = (deadline, status) => {
    if (status === STATUSES.DONE) return 'DONE';
    
    const deadlineDate = new Date(deadline);
    const now = new Date();
    // Saati yok saymak için tarihleri aynı güne sıfırla
    deadlineDate.setHours(0, 0, 0, 0);
    now.setHours(0, 0, 0, 0);

    const diffTime = deadlineDate - now;
    const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));

    if (diffDays < 0) return 'OVERDUE';
    if (diffDays <= 3) return 'CRITICAL';
    if (diffDays >= 10) return 'RELAXED';
    return 'STANDARD';
};


// Tarihe göre kart rengini belirler (Sol Kenarlık Rengi)
const getDateColor = (dateString, status) => {
    const urgency = getUrgencyLevel(dateString, status);
    
    switch (urgency) {
        case 'DONE':
            return { badge: 'bg-green-500 border-green-700 text-white shadow-md', border: 'border-green-500' };
        case 'OVERDUE':
            return { badge: 'bg-red-500 border-red-700 text-white shadow-lg', border: 'border-red-500' };
        case 'CRITICAL':
            return { badge: 'bg-orange-400 border-orange-600 text-gray-900 shadow-md', border: 'border-orange-400' };
        case 'RELAXED':
            return { badge: 'bg-blue-400 border-blue-600 text-white', border: 'border-blue-400' };
        case 'STANDARD':
        default:
            return { badge: 'bg-yellow-400 border-yellow-600 text-gray-900', border: 'border-yellow-400' };
    }
};

const formatUserId = (id) => `${id.substring(0, 4)}...${id.substring(id.length - 4)}`;

// --- Firebase ve Kimlik Doğrulama Mantığı ---

const setupFirebase = () => {
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
    const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

    if (Object.keys(firebaseConfig).length === 0) {
        console.error("Firebase yapılandırması eksik.");
        return { db: null, auth: null, appId: appId };
    }

    try {
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        return { db, auth, appId };
    } catch (e) {
        console.error("Firebase başlatılırken hata oluştu:", e);
        return { db: null, auth: null, appId: appId };
    }
};

const authAndInit = async (auth) => {
    if (!auth) return null;

    const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

    try {
        if (initialAuthToken) {
            await signInWithCustomToken(auth, initialAuthToken);
        } else {
            await signInAnonymously(auth);
        }
    } catch (error) {
        // Ağ hatası gibi durumlarda anonim olarak oturum açmayı denemeye devam et
        console.warn("Özel kimlik doğrulama hatası:", error.message);
        try {
            await signInAnonymously(auth);
        } catch (anonError) {
            console.error("Anonim oturum açma da başarısız oldu:", anonError);
            return null;
        }
    }

    return new Promise(resolve => {
        const unsubscribe = onAuthStateChanged(auth, (user) => {
            if (user) {
                unsubscribe();
                resolve(user);
            }
        });
    });
};

// --- API Fonksiyonları ---

const callGeminiApi = async (systemPrompt, userQuery, schema = null) => {
    const payload = {
        contents: [{ parts: [{ text: userQuery }] }],
        systemInstruction: { parts: [{ text: systemPrompt }] },
    };

    if (schema) {
        payload.generationConfig = {
            responseMimeType: "application/json",
            responseSchema: schema
        };
    }

    // Basit bir yeniden deneme mekanizması (Exponential Backoff)
    for (let i = 0; i < 3; i++) {
        try {
            const response = await fetch(`${GEMINI_API_URL}?key=${API_KEY}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (response.ok) {
                const result = await response.json();
                const jsonText = result.candidates?.[0]?.content?.parts?.[0]?.text;
                if (jsonText && schema) {
                    return JSON.parse(jsonText);
                }
                return jsonText;
            } else {
                console.error(`Gemini API hatası: ${response.status} - ${response.statusText}`);
            }
        } catch (error) {
            console.error(`Gemini API çağrısı sırasında hata oluştu (Deneme ${i + 1}):`, error);
        }
        await new Promise(res => setTimeout(res, Math.pow(2, i) * 1000)); // Gecikme
    }
    return null;
};


// --- Yerel Bildirim Mantığı ---

const requestNotificationPermission = () => {
    if (!("Notification" in window)) {
        console.log("Bu tarayıcı bildirimleri desteklemiyor.");
        return;
    }

    if (Notification.permission !== "granted") {
        Notification.requestPermission().then(permission => {
            if (permission === "granted") {
                console.log("Bildirim izni verildi.");
            } else {
                console.log("Bildirim izni reddedildi.");
            }
        });
    }
};

const showNotification = (title, body, tag) => {
    if (Notification.permission === "granted") {
        new Notification(title, {
            body: body,
            icon: 'https://placehold.co/100x100/dc2626/ffffff?text=Task+Alert',
            tag: tag,
            vibrate: [200, 100, 200]
        });
    }
};


// --- Bileşenler ---

const ConfirmationModal = ({ onConfirm, onCancel, title, message }) => (
    <div className="fixed inset-0 z-[60] overflow-y-auto bg-black bg-opacity-70 flex justify-center items-center p-4">
        <div className="bg-white rounded-xl shadow-2xl w-full max-w-sm p-6 transform transition-all duration-300 scale-100">
            <h3 className="text-xl font-bold text-red-700 mb-4">{title}</h3>
            <p className="text-gray-600 mb-6">{message}</p>
            <div className="flex justify-end space-x-3">
                <button
                    onClick={onCancel}
                    className="px-4 py-2 bg-gray-200 text-gray-700 font-medium rounded-lg shadow hover:bg-gray-300 transition duration-150"
                >
                    Vazgeç
                </button>
                <button
                    onClick={onConfirm}
                    className="px-4 py-2 bg-red-600 text-white font-medium rounded-lg shadow hover:bg-red-700 transition duration-150"
                >
                    Evet, Sil
                </button>
            </div>
        </div>
    </div>
);


// YENİ: Aktivite Akışı Bileşeni
const ActivityStream = React.memo(({ db, task, userId }) => {
    const [activities, setActivities] = useState([]);
    const activityCollectionPath = useMemo(() => 
        `artifacts/${db.appId}/users/${userId}/tasks/${task.id}/activity`, 
    [db.appId, task.id, userId]);

    useEffect(() => {
        if (db) {
            const activityRef = collection(db, activityCollectionPath);
            // Tarihe göre ters sırada getir (En yeni en üstte)
            const q = query(activityRef); 

            const unsubscribe = onSnapshot(q, (snapshot) => {
                const fetchedActivities = snapshot.docs.map(doc => {
                    const data = doc.data();
                    return {
                        id: doc.id,
                        ...data,
                        timestamp: new Date(data.timestamp).toLocaleString('tr-TR'),
                        details: data.details ? JSON.parse(data.details) : {},
                    };
                });
                // Firestore'da orderBy kullanmadığımız için istemci tarafında sıralama
                setActivities(fetchedActivities.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp)));
            }, (error) => {
                console.error("Aktiviteleri dinlerken hata:", error);
            });
            return () => unsubscribe();
        }
    }, [db, activityCollectionPath]);
    
    // Yardımcı fonksiyon: Aktivite metnini oluşturur
    const renderActivityText = (activity) => {
        const { actionType, details } = activity;
        const user = `<span class="font-bold text-indigo-700">${formatUserId(activity.userId)}</span>`;

        try {
            switch (actionType) {
                case 'STATUS_CHANGE':
                    return `${user} görevin durumunu <span class="font-bold text-red-500">${details.oldStatus}</span>'tan <span class="font-bold text-green-600">${details.newStatus}</span>'a değiştirdi.`;
                case 'FIELD_UPDATE':
                    const fieldName = {
                        'priority': 'Öncelik',
                        'deadline': 'Son Tarih',
                        'title': 'Başlık',
                        'description': 'Açıklama',
                        'estimateHours': 'Tahmini Süre',
                        'tags': 'Etiketler'
                    }[details.field] || details.field;
                    // Uzun değerleri kes
                    const oldValue = (details.oldValue || 'boş').toString().substring(0, 30);
                    const newValue = (details.newValue || 'boş').toString().substring(0, 30);
                    return `${user} görevin **${fieldName}** alanını güncelledi. (Eski: ${oldValue}, Yeni: ${newValue})`;
                case 'SUBTASK_ADDED':
                    return `${user} yeni alt görev ekledi: <span class="italic">"${details.text}"</span>`;
                case 'SUBTASK_COMPLETED':
                    return `${user} bir alt görevi <span class="font-bold text-green-600">tamamladı</span>.`;
                case 'SUBTASK_UNCOMPLETED':
                    return `${user} bir alt görevin <span class="font-bold text-orange-500">tamamlanmasını geri aldı</span>.`;
                case 'SUBTASK_DELETED':
                    return `${user} bir alt görevi sildi.`;
                case 'LLM_STEPS_CONVERTED':
                    return `${user} YZ tarafından önerilen <span class="font-bold">${details.count} adımı</span> kontrol listesine dönüştürdü.`;
                case 'TASK_CREATED':
                    return `${user} bu görevi oluşturdu.`;
                case 'COMMENT_ADDED':
                    return `${user} bir yorum ekledi: <span class="italic">"${details.snippet}"</span>`;
                default:
                    return `${user} bilinmeyen bir eylem gerçekleştirdi: ${actionType}`;
            }
        } catch (e) {
            console.error("Aktivite detaylarını ayrıştırırken hata:", e);
            return `${user} bir eylem gerçekleştirdi. (Detaylar ayrıştırılamadı)`;
        }
    };


    return (
        <div className="space-y-3 max-h-96 overflow-y-auto pt-1 bg-white">
            {activities.map((activity) => (
                <div key={activity.id} className="p-3 bg-indigo-50 border border-indigo-200 rounded-lg text-sm shadow-sm">
                    <p 
                        className="text-gray-700 leading-relaxed"
                        dangerouslySetInnerHTML={{ __html: renderActivityText(activity) }}
                    />
                    <span className="block text-xs text-indigo-500 mt-1">{activity.timestamp}</span>
                </div>
            ))}
            {activities.length === 0 && <p className="text-center text-gray-500 text-sm py-4">Bu görev için henüz aktivite yok.</p>}
        </div>
    );
});


// Yorumlar Bölümü (TaskDetailModal içine alındı ve logTaskActivity prop'u eklendi)
const CommentSection = React.memo(({ db, task, userId, logTaskActivity }) => {
    const [comments, setComments] = useState([]);
    const [newCommentText, setNewCommentText] = useState('');
    const commentCollectionPath = useMemo(() => 
        `artifacts/${db.appId}/users/${userId}/tasks/${task.id}/comments`, 
    [db.appId, task.id, userId]);

    useEffect(() => {
        if (db && userId) {
            const commentRef = collection(db, commentCollectionPath);
            const q = query(commentRef);

            const unsubscribe = onSnapshot(q, (snapshot) => {
                const fetchedComments = snapshot.docs.map(doc => ({
                    id: doc.id,
                    ...doc.data(),
                    createdAt: doc.data().createdAt ? new Date(doc.data().createdAt).toISOString() : new Date().toISOString(),
                }));
                // Yorumları oluşturulma tarihine göre tersten sırala (En yeni en üstte)
                setComments(fetchedComments.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt)));
            }, (error) => {
                console.error("Yorumları dinlerken hata:", error);
            });
            return () => unsubscribe();
        }
    }, [db, userId, commentCollectionPath]);

    const handleAddComment = async (e) => {
        e.preventDefault();
        if (newCommentText.trim() === '') return;
        
        const commentText = newCommentText.trim();
        try {
            await addDoc(collection(db, commentCollectionPath), {
                text: commentText,
                userId: userId, // Yorumu yapan kullanıcı ID'si
                createdAt: new Date().toISOString(),
            });

            // Aktiviteyi logla
            await logTaskActivity(task.id, 'COMMENT_ADDED', { snippet: commentText.substring(0, 50) + (commentText.length > 50 ? '...' : '') });
            
            setNewCommentText('');
        } catch (e) {
                console.error("Yorum eklenirken hata:", e);
        }
    };

    return (
        <div className="mt-0 p-0 border border-gray-200 rounded-xl bg-white shadow-inner">
            <form onSubmit={handleAddComment} className="mb-4 sticky top-0 bg-white p-4 rounded-t-xl z-10">
                <textarea
                    value={newCommentText}
                    onChange={(e) => setNewCommentText(e.target.value)}
                    placeholder="Bir yorum yazın..."
                    className="w-full p-3 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 transition duration-150 shadow-sm resize-none"
                    rows="2"
                ></textarea>
                <button
                    type="submit"
                    className="mt-2 w-full px-4 py-2 bg-blue-600 text-white font-medium rounded-lg shadow hover:bg-blue-700 transition duration-150"
                >
                    Yorum Ekle
                </button>
            </form>

            <div className="space-y-4 p-4 max-h-72 overflow-y-auto">
                {comments.map((comment) => (
                    <div key={comment.id} className="p-3 bg-gray-50 border border-gray-200 rounded-lg shadow-sm">
                        <p className="text-gray-800 text-sm mb-1 whitespace-pre-wrap">{comment.text}</p>
                        <div className="flex justify-between items-center text-xs text-gray-500">
                            <span>{formatUserId(comment.userId)}</span>
                            <span>{new Date(comment.createdAt).toLocaleString('tr-TR')}</span>
                        </div>
                    </div>
                ))}
                {comments.length === 0 && <p className="text-center text-gray-500 text-sm py-4">Henüz yorum yok.</p>}
            </div>
        </div>
    );
});


const TaskDetailModal = React.memo(({ task, db, onClose, userId, handleUpdateTaskStatus, onDeleteTask, handleUpdateTaskField, logTaskActivity }) => {
    const [subtasks, setSubtasks] = useState([]);
    const [newSubtaskText, setNewSubtaskText] = useState('');
    const [isLoading, setIsLoading] = useState(false);
    const [llmBreakdown, setLlmBreakdown] = useState(task.llmBreakdown || []);
    const [showConfirmModal, setShowConfirmModal] = useState(false); 
    // YENİ: Sağ panel için aktif sekme durumu
    const [activeTab, setActiveTab] = useState('management'); // 'management' | 'comments' | 'activity'

    const subtaskCollectionPath = useMemo(() => 
        `artifacts/${db.appId}/users/${userId}/tasks/${task.id}/subtasks`, 
    [db.appId, task.id, userId]);

    // Alt Görevleri Gerçek Zamanlı Dinle
    useEffect(() => {
        if (db && userId) {
            const subtaskRef = collection(db, subtaskCollectionPath);
            const unsubscribe = onSnapshot(subtaskRef, (snapshot) => {
                const fetchedSubtasks = snapshot.docs.map(doc => ({
                    id: doc.id,
                    ...doc.data(),
                }));
                setSubtasks(fetchedSubtasks);
            }, (error) => {
                console.error("Alt görevleri dinlerken hata:", error);
            });
            return () => unsubscribe();
        }
    }, [db, userId, task.id, subtaskCollectionPath]);

    // Alt Görev Ekle
    const handleAddSubtask = async (text, priority = 'Orta') => {
        if (text.trim() === '') return;
        try {
            await addDoc(collection(db, subtaskCollectionPath), {
                text: text.trim(),
                isCompleted: false,
                priority: priority,
                createdAt: new Date().toISOString(),
            });
            await logTaskActivity(task.id, 'SUBTASK_ADDED', { text: text.trim(), priority }); // Aktivite Loglama
            setNewSubtaskText('');
        } catch (e) {
            console.error("Alt görev eklenirken hata:", e);
        }
    };

    // Alt Görev Durumunu Değiştir
    const handleToggleSubtask = async (subtaskId, isCompleted) => {
        const subtaskToToggle = subtasks.find(st => st.id === subtaskId);
        if (!subtaskToToggle) return;
        
        try {
            const docRef = doc(db, subtaskCollectionPath, subtaskId);
            await updateDoc(docRef, { isCompleted: !isCompleted });
            
            // Aktivite Loglama
            const action = isCompleted ? 'SUBTASK_UNCOMPLETED' : 'SUBTASK_COMPLETED';
            await logTaskActivity(task.id, action, { subtaskId, text: subtaskToToggle.text }); 

        } catch (e) {
            console.error("Alt görev durumu güncellenirken hata:", e);
        }
    };

    // Alt Görevi Sil
    const handleDeleteSubtask = async (subtaskId) => {
        const deletedSubtask = subtasks.find(st => st.id === subtaskId);
        
        try {
            const docRef = doc(db, subtaskCollectionPath, subtaskId);
            await deleteDoc(docRef);
            
            // Aktivite Loglama
            await logTaskActivity(task.id, 'SUBTASK_DELETED', { text: deletedSubtask?.text || 'Bilinmeyen Alt Görev' }); 

        } catch (e) {
            console.error("Alt görev silinirken hata:", e);
        }
    };

    // LLM Tarafından Oluşturulan Alt Adımları Kontrol Listesine Dönüştür
    const handleConvertLlmBreakdown = async () => {
        if (!llmBreakdown || llmBreakdown.length === 0) return;

        try {
            const batch = writeBatch(db);
            const subtaskRef = collection(db, subtaskCollectionPath);

            llmBreakdown.forEach(step => {
                const newDocRef = doc(subtaskRef);
                batch.set(newDocRef, {
                    text: step.step,
                    isCompleted: false,
                    priority: step.priority || 'Orta',
                    createdAt: new Date().toISOString(),
                });
            });
            
            // Alt adımları temizle ve Firestore'daki görevi güncelle
            const taskDocRef = doc(db, `artifacts/${db.appId}/users/${userId}/tasks`, task.id);
            batch.update(taskDocRef, { llmBreakdown: [] });

            await batch.commit();
            
            // Aktivite Loglama
            await logTaskActivity(task.id, 'LLM_STEPS_CONVERTED', { count: llmBreakdown.length });

            setLlmBreakdown([]); // UI'da temizle
        } catch (e) {
            console.error("LLM adımları alt göreve dönüştürülürken hata:", e);
        }
    };


    // LLM ile Alt Adımları Oluştur
    const handleGenerateSubsteps = async () => {
        setIsLoading(true);
        setLlmBreakdown([]);

        const systemPrompt = "Görevin başlığını ve açıklamasını kullanarak, bu görevi tamamlamak için atılması gereken 3 ila 5 adet küçük alt adım listesi oluştur. Her adım için önceliği 'Yüksek', 'Orta' veya 'Düşük' olarak belirle.";
        const userQuery = `Görevin Başlığı: ${task.title}. Görev Açıklaması: ${task.description}`;
        
        const schema = {
            type: "ARRAY",
            items: {
                type: "OBJECT",
                properties: {
                    "step": { "type": "STRING" },
                    "priority": { "type": "STRING", "enum": ["Yüksek", "Orta", "Düşük"] }
                },
                "propertyOrdering": ["step", "priority"]
            }
        };

        const result = await callGeminiApi(systemPrompt, userQuery, schema);

        if (result && Array.isArray(result)) {
            setLlmBreakdown(result);
            // LLM adımlarını Firestore'a kaydet
            await updateDoc(doc(db, `artifacts/${db.appId}/users/${userId}/tasks`, task.id), {
                llmBreakdown: result
            });
        }
        setIsLoading(false);
    };


    // Modal Kapatıldığında LLM adımlarını temizle
    const handleClose = () => {
        // Alt adımlar kontrol listesine dönüştürülmediyse temizle
        if (llmBreakdown.length > 0) {
            updateDoc(doc(db, `artifacts/${db.appId}/users/${userId}/tasks`, task.id), {
                llmBreakdown: []
            });
        }
        onClose();
    };

    if (!task) return null;
    
    // Alt görev tamamlama yüzdesi
    const completedCount = subtasks.filter(st => st.isCompleted).length;
    const totalCount = subtasks.length;
    const progressPercent = totalCount > 0 ? Math.round((completedCount / totalCount) * 100) : 0;
    
    // Aciliyet etiketini al
    const { badge: dateClass, border: dateBorderClass } = getDateColor(task.deadline, task.status);
    const dateStatus = dateClass.includes('red') ? 'SÜRE GEÇTİ' : dateClass.includes('orange') ? 'KRİTİK' : dateClass.includes('blue') ? 'RAHAT' : dateClass.includes('green') ? 'TAMAMLANDI' : 'STANDART';

    // Öncelik rengini al
    const priorityClass = getPriorityColor(task.priority);
    
    // Sağ Sütun İçeriği
    let rightColumnContent;
    switch (activeTab) {
        case 'comments':
            rightColumnContent = (
                <CommentSection 
                    db={db}
                    task={task}
                    userId={userId}
                    logTaskActivity={logTaskActivity} // Loglama fonksiyonunu yoruma ilet
                />
            );
            break;
        case 'activity':
            rightColumnContent = (
                <ActivityStream
                    db={db}
                    task={task}
                    userId={userId}
                />
            );
            break;
        case 'management':
        default:
            rightColumnContent = (
                <div className="bg-white p-4 rounded-xl border border-gray-200 shadow-sm space-y-3">
                    <h3 className="text-lg font-semibold text-gray-800">Görev Yönetimi</h3>
                    
                    {/* Durum Güncelleme */}
                    <label className="block text-sm font-medium text-gray-700 pt-2">Durumu Güncelle</label>
                    <select
                        value={task.status}
                        onChange={(e) => handleUpdateTaskStatus(task.id, e.target.value)}
                        className="w-full p-2 border border-gray-300 rounded-lg shadow-sm focus:ring-blue-500 focus:border-blue-500 transition duration-150"
                    >
                        {Object.values(STATUSES).map(s => (
                            <option key={s} value={s}>{s}</option>
                        ))}
                    </select>

                    {/* Öncelik Güncelleme */}
                    <label className="block text-sm font-medium text-gray-700 pt-2">Öncelik</label>
                    <select
                        value={task.priority || PRIORITIES.MEDIUM}
                        onChange={(e) => handleUpdateTaskField(task.id, 'priority', e.target.value)}
                        className="w-full p-2 border border-gray-300 rounded-lg shadow-sm focus:ring-blue-500 focus:border-blue-500 transition duration-150"
                    >
                        {Object.values(PRIORITIES).map(p => (
                            <option key={p} value={p}>{p}</option>
                        ))}
                    </select>

                    {/* Tahmini Süre Güncelleme */}
                    <label className="block text-sm font-medium text-gray-700 pt-2">Tahmini Süre (Saat)</label>
                    <input
                        type="number"
                        value={task.estimateHours || ''}
                        onChange={(e) => handleUpdateTaskField(task.id, 'estimateHours', e.target.value ? parseInt(e.target.value, 10) : null)}
                        min="0"
                        placeholder="Örn: 5"
                        className="w-full p-2 border border-gray-300 rounded-lg shadow-sm focus:ring-blue-500 focus:border-blue-500 transition duration-150"
                    />
                    
                    <button
                        onClick={handleGenerateSubsteps}
                        disabled={isLoading}
                        className={`w-full flex items-center justify-center px-4 py-2 mt-4 font-medium rounded-lg shadow transition duration-150 ${isLoading ? 'bg-gray-400 text-gray-700 cursor-not-allowed' : 'bg-purple-600 text-white hover:bg-purple-700'}`}
                    >
                        <BotIcon className="w-5 h-5 mr-2"/> 
                        {isLoading ? 'Adımlar Oluşturuluyor...' : 'YZ Alt Adım Oluştur'}
                    </button>

                    <button
                        // Silme işlemi öncesinde onay modalını göster
                        onClick={() => setShowConfirmModal(true)} 
                        className="w-full flex items-center justify-center px-4 py-2 mt-4 font-medium rounded-lg shadow transition duration-150 bg-red-600 text-white hover:bg-red-700"
                    >
                        <TrashIcon className="w-5 h-5 mr-2"/> Görevi Sil
                    </button>
                </div>
            );
            break;
    }


    return (
        <div className="fixed inset-0 z-50 overflow-y-auto bg-black bg-opacity-50 flex justify-center items-start pt-10 px-4">
            <div className="bg-white rounded-xl shadow-2xl w-full max-w-4xl transform transition-all duration-300 scale-100 mb-10">
                
                {/* Modal Başlık ve Kapatma */}
                <div className="p-6 border-b border-gray-200 flex justify-between items-center bg-gray-50 rounded-t-xl">
                    <h2 className="text-3xl font-bold text-gray-900 break-words max-w-[80%]">{task.title}</h2>
                    <button onClick={handleClose} className="p-2 text-gray-400 hover:text-gray-600 transition duration-150 rounded-full hover:bg-gray-100">
                        <XIcon className="w-6 h-6" />
                    </button>
                </div>
                
                <div className="p-6 grid grid-cols-1 lg:grid-cols-3 gap-6">
                    
                    {/* Sol Sütun: Ana Detaylar ve Alt Görevler */}
                    <div className="lg:col-span-2 space-y-6">
                        
                        {/* Genel Bilgiler */}
                        <div className="bg-white p-4 rounded-xl border border-gray-200 shadow-sm space-y-3">
                            <div className="flex flex-wrap gap-2">
                                {/* Öncelik Rozeti */}
                                <span className={`px-3 py-1 text-xs font-semibold rounded-full border ${priorityClass}`}>
                                    <ZapIcon className="w-3 h-3 inline mr-1"/> Öncelik: {task.priority || PRIORITIES.MEDIUM}
                                </span>
                                {/* Son Tarih Rozeti */}
                                <span className={`px-3 py-1 text-xs font-semibold rounded-full border ${dateClass}`}>
                                    Son Tarih: {task.deadline} ({dateStatus})
                                </span>
                                {/* Tahmini Süre Rozeti */}
                                {task.estimateHours > 0 && (
                                    <span className={`px-3 py-1 text-xs font-semibold rounded-full border bg-indigo-100 text-indigo-700 border-indigo-300`}>
                                        <ClockIcon className="w-3 h-3 inline mr-1"/> Tahmin: {task.estimateHours} saat
                                    </span>
                                )}
                                {/* Durum Rozeti */}
                                <span className={`px-3 py-1 text-xs font-semibold rounded-full border ${getStatusBadgeColor(task.status)}`}>
                                    Durum: {task.status}
                                </span>
                            </div>
                            
                            <h3 className="text-lg font-semibold text-gray-800 pt-2">Açıklama</h3>
                            <p className="text-gray-600 whitespace-pre-wrap">{task.description}</p>
                            
                            <h3 className="text-lg font-semibold text-gray-800 pt-2">Etiketler</h3>
                            <div className="flex flex-wrap gap-2">
                                {(task.tags || []).map((tag, index) => (
                                    <span key={index} className="px-3 py-1 bg-blue-500 text-white text-xs font-medium rounded-full shadow-sm">
                                        {tag}
                                    </span>
                                ))}
                            </div>
                        </div>

                        {/* Alt Görevler/Kontrol Listesi */}
                        <div className="bg-white p-4 rounded-xl border border-gray-200 shadow-sm">
                            <h3 className="text-xl font-bold text-gray-800 mb-4 flex justify-between items-center">
                                Alt Görevler ({completedCount}/{totalCount})
                                <span className={`text-sm font-medium ${progressPercent === 100 ? 'text-green-600' : 'text-blue-600'}`}>{progressPercent}% Tamamlandı</span>
                            </h3>

                            {/* İlerleme Çubuğu */}
                            <div className="w-full bg-gray-200 rounded-full h-2.5 mb-4">
                                <div className={`h-2.5 rounded-full transition-all duration-500 ${progressPercent === 100 ? 'bg-green-600' : 'bg-blue-600'}`} style={{ width: `${progressPercent}%` }}></div>
                            </div>

                            {/* LLM Adımları */}
                            {llmBreakdown && llmBreakdown.length > 0 && (
                                <div className="p-3 bg-purple-50 border border-purple-300 rounded-lg mb-4">
                                    <h4 className="font-semibold text-purple-700 flex items-center mb-2">
                                        <BotIcon className="w-4 h-4 mr-1"/> Yapay Zeka Önerileri
                                    </h4>
                                    <ul className="list-disc list-inside text-sm text-purple-600 space-y-1 mb-3">
                                        {llmBreakdown.map((item, index) => (
                                            <li key={index}>**{item.priority}**: {item.step}</li>
                                        ))}
                                    </ul>
                                    <button
                                        onClick={handleConvertLlmBreakdown}
                                        className="w-full px-3 py-2 bg-purple-600 text-white text-sm font-medium rounded-lg shadow-md hover:bg-purple-700 transition duration-150"
                                    >
                                        <CheckIcon className="w-4 h-4 inline mr-1"/> Kontrol Listesine Dönüştür
                                    </button>
                                </div>
                            )}

                            {/* Yeni Alt Görev Formu */}
                            <div className="flex mb-4">
                                <input
                                    type="text"
                                    value={newSubtaskText}
                                    onChange={(e) => setNewSubtaskText(e.target.value)}
                                    placeholder="Yeni alt görev ekle..."
                                    className="flex-grow p-2 border border-gray-300 rounded-l-lg focus:ring-blue-500 focus:border-blue-500 transition duration-150"
                                />
                                <button
                                    onClick={() => handleAddSubtask(newSubtaskText)}
                                    className="px-4 py-2 bg-green-500 text-white rounded-r-lg hover:bg-green-600 transition duration-150 flex items-center"
                                >
                                    <PlusIcon className="w-5 h-5"/>
                                </button>
                            </div>

                            {/* Alt Görev Listesi */}
                            <div className="space-y-2 max-h-64 overflow-y-auto">
                                {subtasks.map((subtask) => (
                                    <div key={subtask.id} className="flex items-center justify-between p-2 bg-gray-50 rounded-lg border border-gray-200">
                                        <div className="flex items-center flex-grow">
                                            <input
                                                type="checkbox"
                                                checked={subtask.isCompleted}
                                                onChange={() => handleToggleSubtask(subtask.id, subtask.isCompleted)}
                                                className="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 mr-3"
                                            />
                                            <span className={`text-sm ${subtask.isCompleted ? 'line-through text-gray-500' : 'text-gray-700'}`}>
                                                {subtask.text}
                                            </span>
                                        </div>
                                        <button
                                            onClick={() => handleDeleteSubtask(subtask.id)}
                                            className="p-1 text-red-400 hover:text-red-600 transition duration-150 rounded-full hover:bg-red-100 ml-2"
                                >
                                            <TrashIcon className="w-4 h-4"/>
                                        </button>
                                    </div>
                                ))}
                                {subtasks.length === 0 && !llmBreakdown.length && <p className="text-center text-gray-500 text-sm">Alt görev ekleyin veya YZ'den oluşturun.</p>}
                            </div>
                        </div>
                        
                    </div>
                    
                    {/* Sağ Sütun: Sekmeli İçerik */}
                    <div className="lg:col-span-1 space-y-4">
                        
                        {/* Sekme Başlıkları */}
                        <div className="flex bg-gray-100 rounded-xl shadow-inner p-1">
                            {/* Yönetim Sekmesi */}
                            <button
                                onClick={() => setActiveTab('management')}
                                className={`flex-1 flex items-center justify-center p-3 text-sm font-medium rounded-lg transition duration-150 ${activeTab === 'management' ? 'bg-white text-indigo-700 shadow-md' : 'text-gray-600 hover:bg-gray-200'}`}
                            >
                                <SettingsIcon className="w-4 h-4 mr-1"/> Yönetim
                            </button>
                            {/* Yorumlar Sekmesi */}
                            <button
                                onClick={() => setActiveTab('comments')}
                                className={`flex-1 flex items-center justify-center p-3 text-sm font-medium rounded-lg transition duration-150 ${activeTab === 'comments' ? 'bg-white text-indigo-700 shadow-md' : 'text-gray-600 hover:bg-gray-200'}`}
                            >
                                <MessageCircleIcon className="w-4 h-4 mr-1"/> Yorumlar
                            </button>
                            {/* Aktivite Sekmesi */}
                            <button
                                onClick={() => setActiveTab('activity')}
                                className={`flex-1 flex items-center justify-center p-3 text-sm font-medium rounded-lg transition duration-150 ${activeTab === 'activity' ? 'bg-white text-indigo-700 shadow-md' : 'text-gray-600 hover:bg-gray-200'}`}
                            >
                                <ActivityIcon className="w-4 h-4 mr-1"/> Aktivite
                            </button>
                        </div>

                        {/* Sekme İçeriği */}
                        {rightColumnContent}
                        
                    </div>
                </div>
            </div>
            
            {/* Görev Silme Onayı Modalı */}
            {showConfirmModal && (
                <ConfirmationModal
                    title="Görevi Silme Onayı"
                    message={`"${task.title}" başlıklı görevi ve tüm alt görevleri/yorumları/aktiviteleri kalıcı olarak silmek istediğinizden emin misiniz? Bu işlem geri alınamaz.`}
                    onCancel={() => setShowConfirmModal(false)}
                    onConfirm={() => {
                        setShowConfirmModal(false); // Onay modalını kapat
                        handleClose(); // Ana detay modalını kapat
                        onDeleteTask(task.id); // Silme işlemini başlat
                    }}
                />
            )}
        </div>
    );
});


/**
 * Yeni Görev Ekleme Formu
 */
const AddTaskForm = React.memo(({ db, userId, onTaskAdded, logTaskActivity }) => {
    const [title, setTitle] = useState('');
    const [description, setDescription] = useState('');
    const [deadline, setDeadline] = useState(new Date().toISOString().split('T')[0]);
    const [tags, setTags] = useState('');
    const [priority, setPriority] = useState(PRIORITIES.MEDIUM); // Yeni: Öncelik
    const [estimateHours, setEstimateHours] = useState(''); // Yeni: Tahmini Süre
    const [isLoading, setIsLoading] = useState(false);
    const [isSuggestingTags, setIsSuggestingTags] = useState(false);
    const [showModal, setShowModal] = useState(false);
    const [validationError, setValidationError] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();
        setValidationError('');
        setIsLoading(true);

        if (!title.trim() || !deadline) {
            setValidationError('Başlık ve Son Tarih alanları boş bırakılamaz.');
            setIsLoading(false);
            return;
        }

        const taskData = {
            title: title.trim(),
            description: description.trim(),
            deadline: deadline,
            status: STATUSES.TODO,
            createdAt: new Date().toISOString(),
            createdBy: userId,
            tags: tags.split(',').map(tag => tag.trim().toLowerCase()).filter(tag => tag.length > 0),
            priority: priority, 
            estimateHours: estimateHours ? parseInt(estimateHours, 10) : null, 
            llmBreakdown: [],
        };

        try {
            const collectionPath = `artifacts/${db.appId}/users/${userId}/tasks`;
            const docRef = await addDoc(collection(db, collectionPath), taskData);
            
            // YENİ: Görev oluşturma aktivitesini logla
            await logTaskActivity(docRef.id, 'TASK_CREATED', { title: taskData.title });
            
            // Formu temizle ve kapat
            setTitle('');
            setDescription('');
            setDeadline(new Date().toISOString().split('T')[0]);
            setTags('');
            setPriority(PRIORITIES.MEDIUM);
            setEstimateHours('');
            setValidationError('');
            setShowModal(false);
            onTaskAdded();
        } catch (e) {
            console.error("Görev eklenirken hata:", e);
            setValidationError('Görev eklenirken bir hata oluştu. Konsolu kontrol edin.');
        } finally {
            setIsLoading(false);
        }
    };
    
    // LLM ile Etiket Öner
    const handleSuggestTags = async () => {
        if (!title.trim() && !description.trim()) {
            setValidationError('Etiket önermek için Başlık veya Açıklama girilmelidir.');
            return;
        }

        setIsSuggestingTags(true);
        setValidationError('');

        const systemPrompt = "Girdiğiniz görev başlığı ve açıklaması için, en fazla 5 adet, virgülle ayrılmış küçük harfli tek kelimelik etiket listesi döndür. Sadece etiketleri içeren bir JSON dizisi döndür.";
        const userQuery = `Başlık: ${title}. Açıklama: ${description}`;
        
        const schema = {
            type: "OBJECT",
            properties: {
                "tags": { 
                    "type": "ARRAY",
                    "items": { "type": "STRING" }
                }
            },
            "propertyOrdering": ["tags"]
        };

        const result = await callGeminiApi(systemPrompt, userQuery, schema);

        if (result && Array.isArray(result.tags)) {
            const suggestedTags = result.tags.join(', ');
            setTags(suggestedTags);
        } else {
            console.warn("YZ etiket önerisi boş veya hatalı formatta döndü.");
        }
        setIsSuggestingTags(false);
    };

    const today = useMemo(() => new Date().toISOString().split('T')[0], []);

    return (
        <>
            <button 
                onClick={() => setShowModal(true)}
                className="w-full md:w-auto flex items-center justify-center px-6 py-3 bg-indigo-600 text-white font-bold rounded-lg shadow-lg hover:bg-indigo-700 transition duration-200 transform hover:scale-[1.02] active:scale-100"
            >
                <PlusIcon className="w-5 h-5 mr-2" /> Yeni Görev Ekle
            </button>
            
            {showModal && (
                <div className="fixed inset-0 z-50 overflow-y-auto bg-black bg-opacity-50 flex justify-center items-start pt-10 px-4">
                    <div className="bg-white rounded-xl shadow-2xl w-full max-w-lg transform transition-all duration-300 scale-100 mb-10">
                        
                        {/* Modal Başlık */}
                        <div className="p-6 border-b border-gray-200 flex justify-between items-center bg-gray-50 rounded-t-xl">
                            <h2 className="text-2xl font-bold text-gray-900">Yeni Görev Oluştur</h2>
                            <button onClick={() => setShowModal(false)} className="p-2 text-gray-400 hover:text-gray-600 transition duration-150 rounded-full hover:bg-gray-100">
                                <XIcon className="w-6 h-6" />
                            </button>
                        </div>
                        
                        {/* Form İçeriği */}
                        <form onSubmit={handleSubmit} className="p-6 space-y-4">
                            {validationError && (
                                <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative text-sm">
                                    {validationError}
                                </div>
                            )}

                            {/* Başlık */}
                            <div>
                                <label htmlFor="title" className="block text-sm font-medium text-gray-700">Başlık *</label>
                                <input
                                    id="title"
                                    type="text"
                                    value={title}
                                    onChange={(e) => setTitle(e.target.value)}
                                    className="mt-1 w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 transition duration-150"
                                    required
                                />
                            </div>

                            {/* Açıklama */}
                            <div>
                                <label htmlFor="description" className="block text-sm font-medium text-gray-700">Açıklama</label>
                                <textarea
                                    id="description"
                                    value={description}
                                    onChange={(e) => setDescription(e.target.value)}
                                    className="mt-1 w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 transition duration-150 resize-none"
                                    rows="3"
                                />
                            </div>

                            {/* Öncelik ve Tahmini Süre Grubu */}
                            <div className='grid grid-cols-2 gap-4'>
                                {/* Öncelik */}
                                <div>
                                    <label htmlFor="priority" className="block text-sm font-medium text-gray-700">Öncelik</label>
                                    <select
                                        id="priority"
                                        value={priority}
                                        onChange={(e) => setPriority(e.target.value)}
                                        className="mt-1 w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 transition duration-150"
                                    >
                                        {Object.values(PRIORITIES).map(p => (
                                            <option key={p} value={p}>{p}</option>
                                        ))}
                                    </select>
                                </div>
                                
                                {/* Tahmini Süre */}
                                <div>
                                    <label htmlFor="estimateHours" className="block text-sm font-medium text-gray-700">Tahmini Süre (Saat)</label>
                                    <input
                                        id="estimateHours"
                                        type="number"
                                        value={estimateHours}
                                        onChange={(e) => setEstimateHours(e.target.value)}
                                        min="0"
                                        placeholder="Örn: 5"
                                        className="mt-1 w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 transition duration-150"
                                    />
                                </div>
                            </div>

                            {/* Son Tarih */}
                            <div>
                                <label htmlFor="deadline" className="block text-sm font-medium text-gray-700">Son Tarih *</label>
                                <input
                                    id="deadline"
                                    type="date"
                                    value={deadline}
                                    onChange={(e) => setDeadline(e.target.value)}
                                    min={today}
                                    className="mt-1 w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 transition duration-150"
                                    required
                                />
                            </div>
                            
                            {/* Etiketler ve YZ Önerisi */}
                            <div>
                                <label htmlFor="tags" className="block text-sm font-medium text-gray-700">Etiketler (Virgülle ayırın)</label>
                                <div className="flex space-x-2 mt-1">
                                    <input
                                        id="tags"
                                        type="text"
                                        value={tags}
                                        onChange={(e) => setTags(e.target.value)}
                                        placeholder="tasarım, frontend, acil"
                                        className="flex-grow p-3 border border-gray-300 rounded-l-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 transition duration-150"
                                    />
                                    <button
                                        type="button"
                                        onClick={handleSuggestTags}
                                        disabled={isSuggestingTags}
                                        className={`px-4 py-2 font-medium text-white rounded-r-lg shadow-md transition duration-150 flex items-center ${isSuggestingTags ? 'bg-gray-400 cursor-not-allowed' : 'bg-purple-600 hover:bg-purple-700'}`}
                                    >
                                        <BotIcon className="w-5 h-5 mr-1" />
                                        {isSuggestingTags ? 'Öneriliyor...' : 'Öner'}
                                    </button>
                                </div>
                            </div>
                            
                            {/* Görev Ekle Butonu */}
                            <button
                                type="submit"
                                disabled={isLoading}
                                className={`w-full flex items-center justify-center px-4 py-3 font-bold rounded-lg shadow-lg transition duration-150 ${isLoading ? 'bg-gray-400 text-gray-700 cursor-not-allowed' : 'bg-indigo-600 text-white hover:bg-indigo-700'}`}
                            >
                                <SaveIcon className="w-6 h-6 mr-2"/> {isLoading ? 'Kaydediliyor...' : 'Görevi Kaydet'}
                            </button>
                        </form>
                    </div>
                </div>
            )}
        </>
    );
});

/**
 * Tek bir görev kartını temsil eder.
 */
const TaskCard = React.memo(({ task, onCardClick, onDragStart, onDragEnd }) => {
    
    // Son Tarih aciliyetine göre sol kenarlık rengi
    const { badge: dateClass, border: dateBorderClass } = getDateColor(task.deadline, task.status);
    
    // Öncelik rozet rengi
    const priorityClass = getPriorityColor(task.priority || PRIORITIES.MEDIUM); 
    
    // Öncelik seviyesine göre kartın arka plan rengi
    const priorityBgClass = getPriorityBgColor(task.priority || PRIORITIES.MEDIUM);

    return (
        // Sürükle-Bırak (Drag-and-Drop) Özellikleri
        <div
            className={`
                ${priorityBgClass} p-4 mb-4 rounded-xl shadow-lg border-l-4
                cursor-pointer transform transition duration-300 ease-in-out hover:shadow-xl hover:scale-[1.01]
                ${task.isDragging ? 'opacity-50 border-dashed' : ''}
                ${dateBorderClass} // Sol kenar çizgisi, son tarih aciliyetini göstermeye devam ediyor
            `}
            draggable="true"
            onDragStart={(e) => onDragStart(e, task.id)}
            onDragEnd={onDragEnd} 
            onClick={() => onCardClick(task)}
        >
            <div className="flex justify-between items-start mb-2">
                <h3 className="font-semibold text-lg text-gray-800 break-words pr-4">{task.title}</h3>
                <span className={`px-2 py-0.5 text-xs font-semibold rounded-full whitespace-nowrap ${dateClass}`}>
                    {task.deadline}
                </span>
            </div>
            
            <p className="text-sm text-gray-600 mb-3 truncate">{task.description}</p>
            
            {/* Öncelik ve Tahmin */}
            <div className="flex items-center justify-between mt-2 mb-3">
                <span className={`px-2 py-0.5 text-xs font-semibold rounded-full ${priorityClass}`}>
                    <ZapIcon className="w-3 h-3 inline mr-1"/> {task.priority || PRIORITIES.MEDIUM}
                </span>
                {(task.estimateHours && task.estimateHours > 0) && (
                    <span className="text-xs text-indigo-600 font-medium flex items-center">
                        <ClockIcon className="w-3 h-3 mr-1"/> {task.estimateHours} saat
                    </span>
                )}
            </div>

            {/* Etiketler */}
            <div className="flex flex-wrap gap-2 mb-2">
                {(task.tags || []).map((tag, index) => (
                    <span key={index} className="px-2 py-0.5 bg-blue-100 text-blue-700 text-xs font-medium rounded-full shadow-sm">
                        {tag}
                    </span>
                ))}
            </div>

            {/* Alt Adım Durumu (LLM breakdown temizlendikten sonra) */}
            {(task.llmBreakdown && task.llmBreakdown.length > 0) && (
                <div className="flex items-center text-xs text-purple-600 mt-2 font-medium">
                    <BotIcon className="w-4 h-4 mr-1"/> YZ Adımları Bekliyor
                </div>
            )}
        </div>
    );
});


/**
 * Kanban Sütununu temsil eder (TODO, IN_PROGRESS, DONE)
 */
const KanbanColumn = React.memo(({ title, status, tasks, onDrop, onDragOver, onCardClick, onDragStart, onDragEnd }) => {
    // getColumnColor fonksiyonunu kullan
    const columnClass = getColumnColor(status);
    
    return (
        <div
            className="flex-shrink-0 w-full sm:w-80 p-4 rounded-xl shadow-xl bg-gray-100 border border-gray-200"
            onDrop={(e) => onDrop(e, status)}
            onDragOver={onDragOver}
        >
            <h2 className={`text-xl font-bold p-3 rounded-lg text-center ${columnClass} mb-4 shadow-md`}>
                {title} ({tasks.length})
            </h2>
            <div className="min-h-[200px] transition-all duration-200">
                {tasks.map(task => (
                    <TaskCard 
                        key={task.id} 
                        task={task} 
                        onCardClick={onCardClick}
                        onDragStart={onDragStart}
                        onDragEnd={onDragEnd}
                    />
                ))}
                {tasks.length === 0 && (
                    <div className="text-center p-8 text-gray-500 border border-dashed border-gray-300 rounded-lg">
                        Buraya {title} görevlerini sürükleyin.
                    </div>
                )}
            </div>
        </div>
    );
});


/**
 * Kritik ve Süresi Geçmiş görevler için kalıcı hatırlatıcı banner
 */
const ReminderBanner = React.memo(({ count }) => {
    if (count === 0) return null;
    
    const message = count === 1 
        ? "🚨 Bir **kritik/süresi geçmiş** göreviniz var!"
        : `🚨 Dikkat! Toplam **${count} kritik/süresi geçmiş** göreviniz bulunuyor. Önceliklendirin!`;

    return (
        <div className="bg-red-100 border-l-4 border-red-500 text-red-800 p-4 rounded-lg shadow-xl mb-6 flex items-center animate-pulse">
            <AlertTriangleIcon className="w-6 h-6 mr-3 flex-shrink-0" />
            <p className="font-semibold text-base" dangerouslySetInnerHTML={{ __html: message }} />
        </div>
    );
});


/**
 * Ana Uygulama Bileşeni
 */
const App = () => {
    const [db, setDb] = useState(null);
    const [auth, setAuth] = useState(null);
    const [userId, setUserId] = useState(null);
    const [tasks, setTasks] = useState([]);
    const [loading, setLoading] = useState(true);
    const [selectedTask, setSelectedTask] = useState(null);

    // Filtreleme Durumları
    const [filterStatus, setFilterStatus] = useState('ALL');
    const [filterTags, setFilterTags] = useState('');
    const [filterOnlyMine, setFilterOnlyMine] = useState(false);
    const [filterUrgency, setFilterUrgency] = useState('ALL');
    
    // Sürükleme Durumu
    const [draggedTaskId, setDraggedTaskId] = useState(null);
    
    // Bildirim durumu
    const [notificationPermission, setNotificationPermission] = useState(Notification.permission);


    // --- YENİ: Aktivite Loglama Fonksiyonu ---
    const logTaskActivity = useCallback(async (taskId, actionType, details = {}) => {
        if (!db || !userId) return;
        try {
            const activityPath = `artifacts/${db.appId}/users/${userId}/tasks/${taskId}/activity`;
            await addDoc(collection(db, activityPath), {
                actionType,
                // JSON.stringify ile karmaşık nesneleri güvenli bir şekilde sakla
                details: JSON.stringify(details), 
                userId,
                timestamp: new Date().toISOString(),
            });
        } catch (e) {
            console.error("Aktivite loglanırken hata:", e);
        }
    }, [db, userId]);


    // --- Bildirim Kontrolü ve Tetikleme ---

    const checkAndNotify = useCallback((currentTasks) => {
        if (notificationPermission !== "granted") return;
        
        // SADECE SÜRESİ GEÇMİŞ (OVERDUE) görevleri kontrol et
        const overdueTasks = currentTasks.filter(task => {
            return getUrgencyLevel(task.deadline, task.status) === 'OVERDUE';
        });

        if (overdueTasks.length > 0) {
            const title = "❗ Acil Görev Hatırlatıcısı";
            let body = `Toplam ${overdueTasks.length} adet süresi geçmiş göreviniz var.`;

            if (overdueTasks.length <= 3) {
                // En fazla 3 görevin başlığını listele
                const titles = overdueTasks.map(t => t.title).join(', ');
                body = `Süresi geçen görevler: ${titles}`;
            } else {
                 body += ` En acil olan ilk 3 görev: ${overdueTasks.slice(0, 3).map(t => t.title).join(', ')}.`;
            }

            // Aynı bildirimin tekrar tekrar gelmesini engellemek için tek bir tag kullan
            showNotification(title, body, 'overdue-task-alert');
        }
    }, [notificationPermission]);


    // Firebase Başlatma ve Kimlik Doğrulama
    useEffect(() => {
        const { db: firestoreDb, auth: firebaseAuth } = setupFirebase();
        if (firestoreDb && firebaseAuth) {
            setDb(firestoreDb);
            setAuth(firebaseAuth);
            
            // Bildirim izni iste
            requestNotificationPermission();

            authAndInit(firebaseAuth).then(user => {
                if (user) {
                    setUserId(user.uid);
                } else {
                    setLoading(false);
                }
            });
        } else {
            setLoading(false);
        }
    }, []);

    // Görevleri Gerçek Zamanlı Dinle
    useEffect(() => {
        if (db && userId) {
            setLoading(true);
            const collectionPath = `artifacts/${db.appId}/users/${userId}/tasks`;
            const tasksRef = collection(db, collectionPath);
            
            const unsubscribe = onSnapshot(tasksRef, (snapshot) => {
                const fetchedTasks = snapshot.docs.map(doc => ({
                    id: doc.id,
                    ...doc.data(),
                    priority: doc.data().priority || PRIORITIES.MEDIUM,
                    estimateHours: doc.data().estimateHours || null, 
                }));
                setTasks(fetchedTasks);
                setLoading(false);
                
                // Görevler güncellendiğinde de bildirimi kontrol et
                checkAndNotify(fetchedTasks); 

            }, (error) => {
                console.error("Görevleri dinlerken hata:", error);
                setLoading(false);
            });
            
            return () => unsubscribe();
        }
    }, [db, userId, checkAndNotify]);

    // Periyodik Bildirim Kontrolü
    useEffect(() => {
        if (notificationPermission === "granted" && tasks.length > 0) {
            const intervalId = setInterval(() => {
                checkAndNotify(tasks); 
            }, 60000); 

            return () => clearInterval(intervalId);
        }
        return undefined;
    }, [tasks, checkAndNotify, notificationPermission]);


    // --- Görev CRUD Fonksiyonları (Loglama Entegre Edildi) ---
    
    const updateTaskStatus = useCallback(async (taskId, newStatus) => {
        const oldTask = tasks.find(t => t.id === taskId);
        if (!db || !userId || !oldTask) return;

        if (oldTask.status === newStatus) return; // Durum değişmediyse loglama

        try {
            const taskRef = doc(db, `artifacts/${db.appId}/users/${userId}/tasks`, taskId);
            await updateDoc(taskRef, { status: newStatus });

            // Log Activity
            await logTaskActivity(taskId, 'STATUS_CHANGE', { 
                oldStatus: oldTask.status, 
                newStatus: newStatus 
            });

        } catch (e) {
            console.error("Görev durumu güncellenirken hata:", e);
        }
    }, [db, userId, logTaskActivity, tasks]);

    const updateTaskField = useCallback(async (taskId, field, value) => {
        const oldTask = tasks.find(t => t.id === taskId);
        if (!db || !userId || !oldTask) return;
        
        // Değer değişmediyse loglama yapma
        if (oldTask[field] === value) return;

        try {
            const taskRef = doc(db, `artifacts/${db.appId}/users/${userId}/tasks`, taskId);
            await updateDoc(taskRef, { [field]: value });

            // Log Activity
            await logTaskActivity(taskId, 'FIELD_UPDATE', { 
                field, 
                oldValue: oldTask[field] || 'boş', 
                newValue: value || 'boş' 
            });
            
        } catch (e) {
            console.error(`Görev alanı (${field}) güncellenirken hata:`, e);
        }
    }, [db, userId, logTaskActivity, tasks]);

    const deleteTask = useCallback(async (taskId) => {
        if (!db || !userId) return;
        try {
            // Yorum, alt görev ve aktivite alt koleksiyonlarını silme
            const commentQuery = query(collection(db, `artifacts/${db.appId}/users/${userId}/tasks/${taskId}/comments`));
            const subtaskQuery = query(collection(db, `artifacts/${db.appId}/users/${userId}/tasks/${taskId}/subtasks`));
            const activityQuery = query(collection(db, `artifacts/${db.appId}/users/${userId}/tasks/${taskId}/activity`));
            
            const commentsSnapshot = await getDocs(commentQuery);
            const subtasksSnapshot = await getDocs(subtaskQuery);
            const activitySnapshot = await getDocs(activityQuery);

            const batch = writeBatch(db);
            
            // Alt koleksiyonları toplu silme
            commentsSnapshot.docs.forEach((doc) => batch.delete(doc.ref));
            subtasksSnapshot.docs.forEach((doc) => batch.delete(doc.ref));
            activitySnapshot.docs.forEach((doc) => batch.delete(doc.ref)); // Aktiviteyi sil

            // Ana görevi silme
            const taskRef = doc(db, `artifacts/${db.appId}/users/${userId}/tasks`, taskId);
            batch.delete(taskRef);

            await batch.commit();

        } catch (e) {
            console.error("Görev silinirken hata:", e);
        }
    }, [db, userId]);


    // --- Sürükle-Bırak İşleyicileri ---
    
    const handleDragStart = useCallback((e, taskId) => {
        e.dataTransfer.setData('taskId', taskId);
        setDraggedTaskId(taskId);
        setTasks(prev => prev.map(t => t.id === taskId ? { ...t, isDragging: true } : t));
    }, []);

    const handleDragEnd = useCallback(() => {
        setDraggedTaskId(null);
        setTasks(prev => prev.map(t => ({ ...t, isDragging: false })));
    }, []);

    const handleDragOver = useCallback((e) => {
        e.preventDefault(); 
    }, []);

    const handleDrop = useCallback((e, newStatus) => {
        e.preventDefault();
        const taskId = e.dataTransfer.getData('taskId');
        
        if (taskId) {
            updateTaskStatus(taskId, newStatus);
        }
    }, [updateTaskStatus]);


    // --- Filtreleme Mantığı ---

    // Kritik veya Süresi Geçmiş görev sayısını hesaplar (Banner için)
    const urgentTaskCount = useMemo(() => {
        return tasks.filter(task => {
            const urgency = getUrgencyLevel(task.deadline, task.status);
            return (urgency === 'CRITICAL' || urgency === 'OVERDUE') && task.status !== STATUSES.DONE;
        }).length;
    }, [tasks]);


    const filteredTasks = useMemo(() => {
        return tasks.filter(task => {
            // 1. Status Filtresi
            if (filterStatus !== 'ALL' && task.status !== filterStatus) return false;

            // 2. Etiket Filtresi (Yazarak Arama)
            const searchTags = filterTags.toLowerCase().trim().split(',').map(t => t.trim()).filter(t => t.length > 0);
            if (searchTags.length > 0) {
                const taskTags = task.tags || [];
                const hasMatch = searchTags.some(searchTag => 
                    taskTags.some(taskTag => taskTag.includes(searchTag))
                );
                if (!hasMatch) return false;
            }

            // 3. Yalnızca Benim Oluşturduklarım Filtresi (Tek kullanıcı modu için bu daha uygun)
            if (filterOnlyMine && task.createdBy !== userId) return false;

            // 4. Aciliyet Filtresi
            if (filterUrgency !== 'ALL') {
                const urgency = getUrgencyLevel(task.deadline, task.status);
                if (filterUrgency === 'CRITICAL_OR_OVERDUE' && (urgency !== 'CRITICAL' && urgency !== 'OVERDUE')) return false;
                if (filterUrgency === 'RELAXED' && urgency !== 'RELAXED') return false;
            }

            return true;
        });
    }, [tasks, filterStatus, filterTags, filterOnlyMine, userId, filterUrgency]);

    // Filtrelenmiş görevleri sütunlara ayır
    const tasksByStatus = useMemo(() => {
        return Object.keys(STATUSES).reduce((acc, statusKey) => {
            acc[STATUSES[statusKey]] = filteredTasks
                .filter(task => task.status === STATUSES[statusKey])
                .sort((a, b) => {
                    // 1. Öncelik Sıralaması (Yüksek > Orta > Düşük)
                    const priorityOrder = { [PRIORITIES.HIGH]: 3, [PRIORITIES.MEDIUM]: 2, [PRIORITIES.LOW]: 1 };
                    const priorityA = priorityOrder[a.priority] || 0;
                    const priorityB = priorityOrder[b.priority] || 0;

                    if (priorityB !== priorityA) {
                        return priorityB - priorityA; // Yüksek öncelik yukarı
                    }

                    // 2. Son Tarihe göre sırala
                    return new Date(a.deadline) - new Date(b.deadline);
                });
            return acc;
        }, {});
    }, [filteredTasks]);
    
    // Başlangıç yükleniyor ekranı
    if (loading || !userId) {
        return (
            <div className="flex items-center justify-center min-h-screen bg-gray-50 text-gray-700 text-xl font-medium">
                <svg className="animate-spin -ml-1 mr-3 h-5 w-5 text-indigo-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
                    <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                Yükleniyor... Kimlik doğrulama bekleniyor.
            </div>
        );
    }

    return (
        <div className="min-h-screen bg-gray-50 p-4 md:p-8">
            
            {/* Başlık ve Görev Ekleme */}
            <header className="flex flex-col md:flex-row justify-between items-center mb-6">
                <h1 className="text-4xl font-extrabold text-gray-900 mb-4 md:mb-0">
                    İş Takip Panosu <span className="text-base text-gray-500 font-normal">({userId.substring(0, 8)}...)</span>
                </h1>
                <AddTaskForm 
                    db={db} 
                    userId={userId} 
                    onTaskAdded={() => console.log('Görev eklendi')} 
                    logTaskActivity={logTaskActivity} // Loglama fonksiyonunu ekle
                />
            </header>

            {/* --- HATIRLATICI BANNER --- */}
            <ReminderBanner count={urgentTaskCount} />
            
            {/* --- Filtre Paneli --- */}
            <div className="bg-white p-4 rounded-xl shadow-lg mb-6 flex flex-wrap gap-4 items-center border border-gray-200">
                <h3 className="font-bold text-gray-700 mr-4">Filtreler:</h3>
                
                {/* 1. Durum Filtresi (Görünüm) */}
                <div>
                    <label htmlFor="filterStatus" className="block text-xs font-medium text-gray-500">Görünüm (Durum)</label>
                    <select
                        id="filterStatus"
                        value={filterStatus}
                        onChange={(e) => setFilterStatus(e.target.value)}
                        className="mt-1 block w-full p-2 border border-gray-300 rounded-lg text-sm shadow-sm"
                    >
                        <option value="ALL">Tüm Durumlar</option>
                        {Object.values(STATUSES).map(s => (
                            <option key={s} value={s}>{s}</option>
                        ))}
                    </select>
                </div>

                {/* 2. Aciliyet Filtresi */}
                <div>
                    <label htmlFor="filterUrgency" className="block text-xs font-medium text-gray-500">Aciliyet</label>
                    <select
                        id="filterUrgency"
                        value={filterUrgency}
                        onChange={(e) => setFilterUrgency(e.target.value)}
                        className="mt-1 block w-full p-2 border border-gray-300 rounded-lg text-sm shadow-sm"
                    >
                        <option value="ALL">Tümü</option>
                        <option value="CRITICAL_OR_OVERDUE">🔴 Kritik/Süresi Geçenler</option>
                        <option value="RELAXED">🔵 Rahat ({'>='}10 gün)</option> 
                    </select>
                </div>
                
                {/* 3. Etiket Arama */}
                <div>
                    <label htmlFor="filterTags" className="block text-xs font-medium text-gray-500">Etiket Ara</label>
                    <input
                        id="filterTags"
                        type="text"
                        value={filterTags}
                        onChange={(e) => setFilterTags(e.target.value)}
                        placeholder="Örn: frontend, acil"
                        className="mt-1 block w-full p-2 border border-gray-300 rounded-lg text-sm shadow-sm"
                    />
                </div>

                {/* 4. Benim Görevlerim Toggle */}
                <div className="self-end mt-4 md:mt-0">
                    <button
                        onClick={() => setFilterOnlyMine(!filterOnlyMine)}
                        className={`px-4 py-2 font-medium rounded-lg shadow transition duration-150 text-sm ${filterOnlyMine ? 'bg-indigo-500 text-white hover:bg-indigo-600' : 'bg-gray-200 text-gray-700 hover:bg-gray-300'}`}
                    >
                        {filterOnlyMine ? '✅ Yalnızca Benimkiler' : '☐ Yalnızca Benimkiler'}
                    </button>
                </div>
            </div>


            {/* Kanban Panosu Sütunları */}
            <div className="flex flex-col sm:flex-row gap-6 overflow-x-auto pb-4">
                {Object.values(STATUSES).map(status => (
                    <KanbanColumn
                        key={status}
                        title={status}
                        status={status}
                        tasks={tasksByStatus[status] || []}
                        onCardClick={setSelectedTask}
                        onDrop={handleDrop}
                        onDragOver={handleDragOver}
                        onDragStart={handleDragStart}
                        onDragEnd={handleDragEnd}
                    />
                ))}
            </div>
            
            {/* Görev Detay Modalı */}
            {selectedTask && (
                <TaskDetailModal 
                    task={selectedTask}
                    db={db}
                    onClose={() => setSelectedTask(null)}
                    userId={userId}
                    handleUpdateTaskStatus={updateTaskStatus}
                    onDeleteTask={deleteTask}
                    handleUpdateTaskField={updateTaskField} 
                    logTaskActivity={logTaskActivity} // Loglama fonksiyonunu ilet
                />
            )}
        </div>
    );
};

export default App;
