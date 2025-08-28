/* =======================================
   src/firebase.js
   ======================================= */
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";
import { getAnalytics } from "firebase/analytics";

const firebaseConfig = {
  apiKey: "AIzaSyCyo1Y7Sl8_z3fNM3iR6tt_jopS5O0UXoA",
  authDomain: "index-50000.firebaseapp.com",
  projectId: "index-50000",
  storageBucket: "index-50000.firebasestorage.app",
  messagingSenderId: "446470628655",
  appId: "1:446470628655:web:c031132f16c59264cfedda",
  measurementId: "G-0C4NZLLC58"
};

const app = initializeApp(firebaseConfig);

export const auth = getAuth(app);
export const db = getFirestore(app);
export const analytics = getAnalytics(app);

/* =======================================
   src/hooks/useAuth.js
   ======================================= */
import { useEffect, useState } from "react";
import { auth, db } from "../firebase";
import {
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  onAuthStateChanged,
  signOut,
  updateProfile
} from "firebase/auth";
import { doc, setDoc, getDoc } from "firebase/firestore";

export function useAuth() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const unsub = onAuthStateChanged(auth, async (firebaseUser) => {
      if (firebaseUser) {
        const ref = doc(db, "users", firebaseUser.uid);
        const snap = await getDoc(ref);

        setUser({
          uid: firebaseUser.uid,
          email: firebaseUser.email,
          name: firebaseUser.displayName || "",
          role: snap.exists() ? snap.data().role : "user",
        });
      } else {
        setUser(null);
      }
    });

    return () => unsub();
  }, []);

  const signup = async (email, password, name) => {
    const res = await createUserWithEmailAndPassword(auth, email, password);
    await updateProfile(res.user, { displayName: name });
    await setDoc(doc(db, "users", res.user.uid), { name, role: "user" });
  };

  const login = (email, password) =>
    signInWithEmailAndPassword(auth, email, password);

  const logout = () => signOut(auth);

  return { user, signup, login, logout };
}

/* =======================================
   src/hooks/useFirestore.js
   ======================================= */
import { db } from "../firebase";
import { doc, setDoc, collection, addDoc } from "firebase/firestore";

export const saveCatalog = async (catalog) => {
  await setDoc(doc(db, "catalog", "main"), catalog);
};

export const saveAFA = async (formData, user) => {
  await addDoc(collection(db, "afa_registrations"), {
    ...formData,
    userId: user.uid,
    createdAt: new Date(),
  });
};stuysl97i2

/* =======================================
   src/components/Header.jsx
   ======================================= */
import React from 'react';
export default function Header({ user, onLogout }) {
  return (
    <header className="flex justify-between items-center p-4 bg-emerald-100">
      <h1 className="text-xl font-bold">Affordable Bundles GH🇬🇭</h1>
      <div>
        {user ? (
          <div className="flex items-center space-x-4">
            <span>{user.name}</span>
            <button onClick={onLogout} className="px-2 py-1 bg-red-500 text-white rounded">
              Logout
            </button>
          </div>
        ) : null}
      </div>
    </header>
  );
}

/* =======================================
   src/components/Footer.jsx
   ======================================= */
import React from 'react';
export default function Footer() {
  return (
    <footer className="p-4 text-center text-sm text-gray-600">
      © 2025 Affordable Bundles GH | Customer Support: 0556429525
    </footer>
  );
}

/* =======================================
   src/components/AuthModal.jsx
   ======================================= */
import React, { useState } from 'react';
export default function AuthModal({ login, signup }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [name, setName] = useState('');
  const [isSignup, setIsSignup] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (isSignup) await signup(email, password, name);
    else await login(email, password);
  };

  return (
    <div className="max-w-md mx-auto bg-white p-6 rounded shadow">
      <h2 className="text-xl font-bold mb-4">{isSignup ? 'Sign Up' : 'Login'}</h2>
      <form onSubmit={handleSubmit} className="space-y-3">
        {isSignup && (
          <input type="text" placeholder="Name" value={name} onChange={e => setName(e.target.value)} className="w-full p-2 border rounded" />
        )}
        <input type="email" placeholder="Email" value={email} onChange={e => setEmail(e.target.value)} className="w-full p-2 border rounded" />
        <input type="password" placeholder="Password" value={password} onChange={e => setPassword(e.target.value)} className="w-full p-2 border rounded" />
        <button type="submit" className="w-full bg-emerald-500 text-white p-2 rounded">{isSignup ? 'Sign Up' : 'Login'}</button>
      </form>
      <p className="text-sm mt-2">
        {isSignup ? 'Already have an account?' : "Don't have an account?"}
        <button className="text-blue-500 ml-1" onClick={() => setIsSignup(!isSignup)}>
          {isSignup ? 'Login' : 'Sign Up'}
        </button>
      </p>
    </div>
  );
}

/* =======================================
   src/components/Dashboard.jsx
   ======================================= */
import React from 'react';
import Bundles from './Bundles';
import AdminPanel from './AdminPanel';
import AFAForm from './AFAForm';
export default function Dashboard({ user, catalog, saveCatalog, saveAFA }) {
  return (
    <div className="space-y-6">
      <Bundles catalog={catalog} />
      <AFAForm user={user} saveAFA={saveAFA} />
      {user.role === 'admin' && <AdminPanel catalog={catalog} saveCatalog={saveCatalog} />}
    </div>
  );
}

/* =======================================
   src/components/Bundles.jsx
   ======================================= */
import React from 'react';
export default function Bundles({ catalog }) {
  return (
    <div>
      <h2 className="text-lg font-bold mb-2">Available Bundles</h2>
      <ul className="space-y-2">
        {catalog?.categories?.map(cat => (
          <li key={cat.name} className="p-3 border rounded">
            <h3 className="font-semibold">{cat.name}</h3>
            <p>{cat.description}</p>
            <a href="https://paystack.shop/pay/stuysl97i2" target="_blank" className="text-blue-600 underline">Buy Now</a>
          </li>
        ))}
      </ul>
    </div>
  );
}

/* =======================================
   src/components/AdminPanel.jsx
   ======================================= */
import React, { useState } from 'react';
export default function AdminPanel({ catalog, saveCatalog }) {
  const [localCatalog, setLocalCatalog] = useState(catalog);
  const handleSave = () => {
    saveCatalog(localCatalog);
    alert('Catalog saved to Firebase!');
  };
  return (
    <div className="p-4 border rounded bg-yellow-50">
      <h2 className="text-lg font-bold mb-2">Admin Panel</h2>
      <button onClick={handleSave} className="px-3 py-1 bg-green-500 text-white rounded">Save Catalog</button>
    </div>
  );
}

/* =======================================
   src/components/AFAForm.jsx
   ======================================= */
import React, { useState } from 'react';
export default function AFAForm({ user, saveAFA }) {
  const [formData, setFormData] = useState({ fullName: '', phone: '', email: user?.email || '' });
  const handleSubmit = async (e) => {
    e.preventDefault();
    await saveAFA(formData, user);
    alert('AFA Registration Submitted!');
    setFormData({ fullName: '', phone: '', email: user?.email || '' });
  };
  return (
    <form onSubmit={handleSubmit} className="p-4 border rounded bg-emerald-50 space-y-2">
      <h2 className="text-lg font-bold">AFA Membership Registration</h2>
      <input type="text" placeholder="Full Name" value={formData.fullName} onChange={e => setFormData({...formData, fullName: e.target.value})} className="w-full p-2 border rounded" required />
      <input type="text" placeholder="Phone" value={formData.phone} onChange={e => setFormData({...formData, phone: e.target.value})} className="w-full p-2 border rounded" required />
      <input type="email" placeholder="Email" value={formData.email} readOnly className="w-full p-2 border rounded bg-gray-200" />
      <button type="submit" className="w-full bg-emerald-500 text-white p-2 rounded">Submit & Pay</button>
    </form>
  );
}

/* =======================================
   src/components/Support.jsx
   ======================================= */
import React from 'react';
export default function Support() {
  return (
    <div className="p-3 border rounded bg-blue-50">
      <h3 className="font-bold">Customer Support</h3>
      <p>Phone/WhatsApp: <a href="tel:+233556429525" className="text-blue-600 underline">0556429525</a></p>
    </div>
  );
}


