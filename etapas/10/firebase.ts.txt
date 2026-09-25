import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';

const firebaseConfig = {
  apiKey: 'AIzaSyBJvYOFOppmjgziU7bFGi92BW3DoihGRsU',
  authDomain: 'aulamobile-1ed38.firebaseapp.com',
  projectId: 'aulamobile-1ed38',
  storageBucket: 'aulamobile-1ed38.firebasestorage.app',
  messagingSenderId: '19702875368',
  appId: '1:19702875368:web:765201fdf67353b18d355d',
};

const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
