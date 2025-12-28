import React, { useState, useEffect, createContext, useContext } from 'react';
import { BrowserRouter as Router, Routes, Route, Link, Navigate } from 'react-router-dom';

/*
  AppointmentBookingApp.jsx
  - Single-file starter React app for an Appointment Booking System.
  - Uses Tailwind CSS classes for styling (assumes Tailwind is configured in the project).
  - Authentication, data storage, and reminders are stubbed so you can hook Firebase / Node.js easily.
  - Default export is the App component. Use this file as a starting point and split into modules as you progress.

*/

// ---------------------------
// Simple in-memory "API" (replace with real backend)
// ---------------------------
const fakeDB = {
  users: [
    { id: 'p1', role: 'patient', name: 'Alice', email: 'alice@example.com', password: 'pass' },
    { id: 'd1', role: 'doctor', name: 'Dr. XX', email: 'drxx@example.com', password: 'pass' },
  ],
  appointments: [],
  posts: [
    { id: 'b1', title: '5 Tips for Healthy Living', content: 'Drink water, sleep well...' },
  ],
  feedback: [],
};

// ---------------------------
// Auth Context (stub) — replace with Firebase Auth
// ---------------------------
const AuthContext = createContext();
function useAuth() {
  return useContext(AuthContext);
}

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // TODO: check localStorage or firebase auth state here
    const stored = localStorage.getItem('abs_user');
    if (stored) setUser(JSON.parse(stored));
  }, []);

  function login(email, password) {
    const found = fakeDB.users.find((u) => u.email === email && u.password === password);
    if (found) {
      setUser(found);
      localStorage.setItem('abs_user', JSON.stringify(found));
      return { ok: true };
    }
    return { ok: false, message: 'Invalid credentials' };
  }

  function logout() {
    setUser(null);
    localStorage.removeItem('abs_user');
  }

  function register(payload) {
    // very basic registration: should validate & hash passwords in real app
    const id = 'p' + (fakeDB.users.length + 1);
    const newUser = { id, role: 'patient', ...payload };
    fakeDB.users.push(newUser);
    setUser(newUser);
    localStorage.setItem('abs_user', JSON.stringify(newUser));
    return { ok: true };
  }

  return (
    <AuthContext.Provider value={{ user, login, logout, register }}>{children}</AuthContext.Provider>
  );
}

// ---------------------------
// Utilities
// ---------------------------
function formatDateTime(dt) {
  const d = new Date(dt);
  return d.toLocaleString();
}

// ---------------------------
// Components
// ---------------------------
function Navbar() {
  const { user, logout } = useAuth();
  return (
    <nav className="bg-white shadow p-4 flex flex-wrap items-center justify-between">
      <div className="flex items-center gap-4">
        <Link to="/" className="font-bold text-xl">ClinicCare</Link>
        <Link to="/blog" className="text-sm">Health Tips</Link>
      </div>
      <div className="flex items-center gap-4">
        {user ? (
          <>
            <span className="hidden sm:inline">Hello, {user.name}</span>
            {user.role === 'doctor' ? <Link to="/doctor" className="px-3 py-1 bg-indigo-600 text-white rounded">Doctor Panel</Link> : <Link to="/dashboard" className="px-3 py-1 bg-indigo-600 text-white rounded">Dashboard</Link>}
            <button onClick={logout} className="px-3 py-1 border rounded">Logout</button>
          </>
        ) : (
          <>
            <Link to="/login" className="px-3 py-1 border rounded">Login</Link>
            <Link to="/register" className="px-3 py-1 bg-green-600 text-white rounded">Register</Link>
          </>
        )}
      </div>
    </nav>
  );
}

function Home() {
  return (
    <div className="p-6">
      <h1 className="text-3xl font-semibold mb-4">Welcome to ClinicCare</h1>
      <p className="mb-4">Book appointments, get reminders, read health tips, and send feedback.</p>
      <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <div className="p-4 border rounded">Fast booking with real-time availability</div>
        <div className="p-4 border rounded">Secure patient dashboard</div>
      </div>
    </div>
  );
}

// ---------------------------
// Auth Pages
// ---------------------------
function LoginPage() {
  const { login } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState(null);

  function handleSubmit(e) {
    e.preventDefault();
    const res = login(email, password);
    if (!res.ok) setError(res.message);
  }

  return (
    <div className="max-w-md mx-auto p-6">
      <h2 className="text-2xl mb-4">Login</h2>
      <form onSubmit={handleSubmit} className="space-y-4">
        <input className="w-full p-2 border rounded" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />
        <input type="password" className="w-full p-2 border rounded" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
        {error && <div className="text-red-600">{error}</div>}
        <button className="w-full py-2 bg-indigo-600 text-white rounded">Login</button>
      </form>
    </div>
  );
}

function RegisterPage() {
  const { register } = useAuth();
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [message, setMessage] = useState(null);

  function handleSubmit(e) {
    e.preventDefault();
    const res = register({ name, email, password });
    if (res.ok) setMessage('Registered & logged in');
  }

  return (
    <div className="max-w-md mx-auto p-6">
      <h2 className="text-2xl mb-4">Create an account</h2>
      <form onSubmit={handleSubmit} className="space-y-4">
        <input className="w-full p-2 border rounded" placeholder="Full name" value={name} onChange={(e) => setName(e.target.value)} />
        <input className="w-full p-2 border rounded" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />
        <input type="password" className="w-full p-2 border rounded" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
        {message && <div className="text-green-600">{message}</div>}
        <button className="w-full py-2 bg-green-600 text-white rounded">Register</button>
      </form>
    </div>
  );
}

// ---------------------------
// Booking Component (patient-facing)
// ---------------------------
function Booking() {
  const { user } = useAuth();
  const [date, setDate] = useState('');
  const [time, setTime] = useState('09:00');
  const [reason, setReason] = useState('');
  const [message, setMessage] = useState(null);

  function checkAvailability(dateTime) {
    // naive check: no other appointment at exact time
    return !fakeDB.appointments.some((a) => a.dateTime === dateTime);
  }

  function handleBook(e) {
    e.preventDefault();
    if (!user || user.role !== 'patient') return setMessage('You must be logged in as a patient to book.');
    const dateTime = new Date(date + 'T' + time).toISOString();
    if (!checkAvailability(dateTime)) return setMessage('Slot not available');
    const appt = {
      id: 'a' + (fakeDB.appointments.length + 1),
      patientId: user.id,
      patientName: user.name,
      doctorId: 'd1',
      dateTime,
      reason,
      status: 'confirmed',
      createdAt: new Date().toISOString(),
    };
    fakeDB.appointments.push(appt);
    setMessage('Appointment confirmed for ' + formatDateTime(dateTime));
    // in real app: call backend, send confirmation email, schedule reminder
  }

  return (
    <div className="max-w-lg mx-auto p-6">
      <h2 className="text-xl mb-4">Book an appointment</h2>
      <form onSubmit={handleBook} className="space-y-3">
        <label className="block">Date</label>
        <input type="date" className="w-full p-2 border rounded" value={date} onChange={(e) => setDate(e.target.value)} required />
        <label className="block">Time</label>
        <input type="time" className="w-full p-2 border rounded" value={time} onChange={(e) => setTime(e.target.value)} required />
        <label className="block">Reason</label>
        <input className="w-full p-2 border rounded" value={reason} onChange={(e) => setReason(e.target.value)} />
        <button className="w-full py-2 bg-indigo-600 text-white rounded">Confirm Booking</button>
      </form>
      {message && <div className="mt-4 p-3 bg-green-50 border rounded">{message}</div>}
    </div>
  );
}

// ---------------------------
// Patient Dashboard
// ---------------------------
function PatientDashboard() {
  const { user } = useAuth();
  const [appts, setAppts] = useState([]);

  useEffect(() => {
    if (!user) return;
    const my = fakeDB.appointments.filter((a) => a.patientId === user.id).sort((a, b) => new Date(a.dateTime) - new Date(b.dateTime));
    setAppts(my);
  }, [user]);

  if (!user) return <Navigate to="/login" />;

  return (
    <div className="p-6 max-w-3xl mx-auto">
      <h2 className="text-2xl mb-4">Your Dashboard</h2>
      <div className="mb-6">
        <h3 className="font-semibold">Upcoming appointments</h3>
        {appts.length === 0 ? (
          <div>No appointments. <Link to="/book" className="text-indigo-600">Book one</Link>.</div>
        ) : (
          <ul className="space-y-2">
            {appts.map((a) => (
              <li key={a.id} className="p-3 border rounded flex justify-between items-center">
                <div>
                  <div className="font-medium">{formatDateTime(a.dateTime)}</div>
                  <div className="text-sm text-gray-600">{a.reason}</div>
                </div>
                <div className="text-sm">Status: {a.status}</div>
              </li>
            ))}
          </ul>
        )}
      </div>

      <div>
        <h3 className="font-semibold">Consultation history</h3>
        <div className="text-sm text-gray-600">(In a real app, doctor's notes & prescriptions would appear here.)</div>
      </div>
    </div>
  );
}

// ---------------------------
// Doctor/Admin Panel
// ---------------------------
function DoctorPanel() {
  const { user } = useAuth();
  const [appointments, setAppointments] = useState([]);
  const [selected, setSelected] = useState(null);
  const [note, setNote] = useState('');

  useEffect(() => {
    setAppointments(fakeDB.appointments.filter((a) => a.doctorId === 'd1').sort((a, b) => new Date(a.dateTime) - new Date(b.dateTime)));
  }, []);

  if (!user || user.role !== 'doctor') return <Navigate to="/login" />;

  function saveNote() {
    if (!selected) return;
    // attach note to appointment (in real app, write to DB)
    selected.doctorNote = note;
    setNote('');
    // force refresh
    setAppointments([...appointments]);
  }

  function updateStatus(appt, status) {
    appt.status = status;
    setAppointments([...appointments]);
  }

  return (
    <div className="p-6 max-w-4xl mx-auto">
      <h2 className="text-2xl mb-4">Doctor Panel</h2>
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-4">
        <div className="lg:col-span-2">
          <h3 className="font-semibold mb-2">Upcoming Appointments</h3>
          <ul className="space-y-2">
            {appointments.map((a) => (
              <li key={a.id} className="p-3 border rounded flex justify-between">
                <div>
                  <div className="font-medium">{formatDateTime(a.dateTime)}</div>
                  <div className="text-sm">Patient: {a.patientName}</div>
                </div>
                <div className="text-right">
                  <div className="mb-2">{a.status}</div>
                  <div className="flex gap-2">
                    <button onClick={() => { setSelected(a); setNote(a.doctorNote || ''); }} className="px-2 py-1 border rounded">Open</button>
                    <button onClick={() => updateStatus(a, 'completed')} className="px-2 py-1 bg-green-600 text-white rounded">Complete</button>
                    <button onClick={() => updateStatus(a, 'canceled')} className="px-2 py-1 border rounded">Cancel</button>
                  </div>
                </div>
              </li>
            ))}
          </ul>
        </div>

        <aside className="p-3 border rounded">
          <h4 className="font-semibold">Patient details & Notes</h4>
          {selected ? (
            <div>
              <div className="mt-2">Name: {selected.patientName}</div>
              <div className="mt-2">Time: {formatDateTime(selected.dateTime)}</div>
              <textarea value={note} onChange={(e) => setNote(e.target.value)} className="w-full p-2 border rounded mt-2" placeholder="Write consultation notes here..." />
              <button onClick={saveNote} className="mt-2 w-full py-2 bg-indigo-600 text-white rounded">Save Note</button>
            </div>
          ) : (
            <div className="text-sm text-gray-600">Select an appointment to view patient info and add notes.</div>
          )}
        </aside>
      </div>
    </div>
  );
}

// ---------------------------
// Blog / Health Tips with simple CMS for admin
// ---------------------------
function BlogList() {
  const [posts, setPosts] = useState(fakeDB.posts);
  return (
    <div className="p-6 max-w-3xl mx-auto">
      <h2 className="text-2xl mb-4">Health Tips & Blog</h2>
      <ul className="space-y-4">
        {posts.map((p) => (
          <li key={p.id} className="p-4 border rounded">
            <h3 className="font-semibold">{p.title}</h3>
            <p className="text-sm text-gray-700">{p.content}</p>
          </li>
        ))}
      </ul>
    </div>
  );
}

function BlogAdmin() {
  const { user } = useAuth();
  const [posts, setPosts] = useState(fakeDB.posts);
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  if (!user || user.role !== 'doctor') return <Navigate to="/login" />;

  function addPost() {
    const id = 'b' + (posts.length + 1);
    const newPost = { id, title, content };
    fakeDB.posts.push(newPost);
    setPosts([...fakeDB.posts]);
    setTitle('');
    setContent('');
  }

  function removePost(id) {
    fakeDB.posts = fakeDB.posts.filter((p) => p.id !== id);
    setPosts([...fakeDB.posts]);
  }

  return (
    <div className="p-6 max-w-3xl mx-auto">
      <h2 className="text-2xl mb-4">Manage Blog Posts</h2>
      <div className="space-y-3">
        <input className="w-full p-2 border rounded" placeholder="Title" value={title} onChange={(e) => setTitle(e.target.value)} />
        <textarea className="w-full p-2 border rounded" placeholder="Content" value={content} onChange={(e) => setContent(e.target.value)} />
        <button onClick={addPost} className="px-4 py-2 bg-indigo-600 text-white rounded">Add Post</button>
      </div>
      <div className="mt-6">
        <h3 className="font-semibold">Existing Posts</h3>
        <ul className="space-y-2 mt-2">
          {posts.map((p) => (
            <li key={p.id} className="p-3 border rounded flex justify-between">
              <div>
                <div className="font-medium">{p.title}</div>
                <div className="text-sm">{p.content.slice(0, 80)}...</div>
              </div>
              <button onClick={() => removePost(p.id)} className="px-2 py-1 border rounded">Delete</button>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}

// ---------------------------
// Feedback Page
// ---------------------------
function Feedback() {
  const { user } = useAuth();
  const [text, setText] = useState('');
  const [message, setMessage] = useState(null);

  function submit() {
    fakeDB.feedback.push({ id: 'f' + (fakeDB.feedback.length + 1), user: user ? user.id : 'anon', text, createdAt: new Date().toISOString() });
    setText('');
    setMessage('Thanks for your feedback!');
  }

  return (
    <div className="p-6 max-w-2xl mx-auto">
      <h2 className="text-2xl mb-4">Feedback</h2>
      <textarea value={text} onChange={(e) => setText(e.target.value)} className="w-full p-2 border rounded" placeholder="How was your consultation?" />
      <button onClick={submit} className="mt-3 px-4 py-2 bg-indigo-600 text-white rounded">Submit</button>
      {message && <div className="mt-3 text-green-600">{message}</div>}
    </div>
  );
}

// ---------------------------
// Simple Reminders Scheduler (client-side stub)
// In production, use server / cloud functions to send SMS / email at scheduled times.
// ---------------------------
function ReminderTester() {
  // shows how reminders would be scheduled
  function scheduleReminder(apptId, minutesBefore = 60) {
    const appt = fakeDB.appointments.find((a) => a.id === apptId);
    if (!appt) return;
    const when = new Date(appt.dateTime).getTime() - minutesBefore * 60 * 1000;
    console.log('Reminder scheduled at', new Date(when).toISOString());
    // On server: enqueue job to send email/SMS at `when`.
  }

  return (
    <div className="p-6 max-w-2xl mx-auto">
      <h3 className="font-semibold">Reminders (demo)</h3>
      <p className="text-sm text-gray-600">Open browser console to see scheduled reminder timestamps after booking an appointment.</p>
      <ul className="mt-3">
        {fakeDB.appointments.map((a) => (
          <li key={a.id} className="flex justify-between p-2 border rounded mb-2">
            <div>{a.patientName} — {formatDateTime(a.dateTime)}</div>
            <button onClick={() => scheduleReminder(a.id)} className="px-2 py-1 border rounded">Schedule reminder</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

// ---------------------------
// Main App & Routes
// ---------------------------
export default function App() {
  return (
    <AuthProvider>
      <Router>
        <div className="min-h-screen bg-gray-50">
          <Navbar />
          <main className="py-6">
            <Routes>
              <Route path="/" element={<Home />} />
              <Route path="/login" element={<LoginPage />} />
              <Route path="/register" element={<RegisterPage />} />
              <Route path="/book" element={<Booking />} />
              <Route path="/dashboard" element={<PatientDashboard />} />
              <Route path="/doctor" element={<DoctorPanel />} />
              <Route path="/blog" element={<BlogList />} />
              <Route path="/manage-blog" element={<BlogAdmin />} />
              <Route path="/feedback" element={<Feedback />} />
              <Route path="/reminders" element={<ReminderTester />} />
              <Route path="*" element={<div className="p-6">Page not found</div>} />
            </Routes>
          </main>
        </div>
      </Router>
    </AuthProvider>
  );
}

