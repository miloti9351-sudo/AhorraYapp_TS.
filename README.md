{
  "name": "ahorrayaapp",
  "version": "1.0.0",
  "main": "node_modules/expo/AppEntry.js",
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web"
  },
  "dependencies": {
    "@react-native-async-storage/async-storage": "^1.19.3",
    "@react-native-firebase/app": "^17.0.0",
    "@react-native-firebase/auth": "^17.0.0",
    "@react-native-firebase/firestore": "^17.0.0",
    "@react-navigation/native": "^6.1.0",
    "@react-navigation/stack": "^6.3.0",
    "expo": "^48.0.0",
    "expo-location": "^15.0.0",
    "expo-status-bar": "~1.4.0",
    "react": "18.2.0",
    "react-native": "0.71.0",
    "react-native-maps": "^1.7.1",
    "react-native-qrcode-svg": "^6.2.0",
    "react-native-screens": "~3.20.0",
    "react-native-vector-icons": "^10.0.0"
  },
  "devDependencies": {
    "@babel/core": "^7.20.0"
  }
}
import React from 'react';
import { StatusBar } from 'expo-status-bar';
import { StyleSheet } from 'react-native';
import AppNavigator from './src/navigation/AppNavigator';
import { AuthProvider } from './src/utils/AuthContext';

export default function App() {
  return (
    <AuthProvider>
      <StatusBar style="auto" />
      <AppNavigator />
    </AuthProvider>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
});
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

// Configuración de Firebase - REEMPLAZA CON TUS DATOS
const firebaseConfig = {
  apiKey: "tu-api-key",
  authDomain: "tu-proyecto.firebaseapp.com",
  projectId: "tu-proyecto-id",
  storageBucket: "tu-proyecto.appspot.com",
  messagingSenderId: "123456789",
  appId: "tu-app-id"
};

// Inicializar Firebase
const app = initializeApp(firebaseConfig);

// Exportar servicios
export const auth = getAuth(app);
export const db = getFirestore(app);
export default app;
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import Ionicons from 'react-native-vector-icons/Ionicons';

// Screens
import LoginScreen from '../screens/LoginScreen';
import MapScreen from '../screens/MapScreen';
import OfferDetailScreen from '../screens/OfferDetailScreen';
import ProfileScreen from '../screens/ProfileScreen';
import CouponsScreen from '../screens/CouponsScreen';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

function MainTabs() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName;
          
          if (route.name === 'Mapa') {
            iconName = focused ? 'map' : 'map-outline';
          } else if (route.name === 'Cupones') {
            iconName = focused ? 'ticket' : 'ticket-outline';
          } else if (route.name === 'Perfil') {
            iconName = focused ? 'person' : 'person-outline';
          }
          
          return <Ionicons name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#4e54c8',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tab.Screen name="Mapa" component={MapScreen} />
      <Tab.Screen name="Cupones" component={CouponsScreen} />
      <Tab.Screen name="Perfil" component={ProfileScreen} />
    </Tab.Navigator>
  );
}

export default function AppNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen 
          name="Login" 
          component={LoginScreen}
          options={{ headerShown: false }}
        />
        <Stack.Screen 
          name="Main" 
          component={MainTabs}
          options={{ headerShown: false }}
        />
        <Stack.Screen 
          name="OfferDetail" 
          component={OfferDetailScreen}
          options={{ title: 'Detalle de Oferta' }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
import React, { createContext, useState, useContext, useEffect } from 'react';
import { onAuthStateChanged, signInWithEmailAndPassword, createUserWithEmailAndPassword, signOut } from 'firebase/auth';
import { auth } from '../config/firebase';

const AuthContext = createContext();

export const useAuth = () => {
  return useContext(AuthContext);
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (user) => {
      setUser(user);
      setLoading(false);
    });

    return unsubscribe;
  }, []);

  const login = async (email, password) => {
    try {
      const userCredential = await signInWithEmailAndPassword(auth, email, password);
      return { success: true, user: userCredential.user };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const register = async (email, password) => {
    try {
      const userCredential = await createUserWithEmailAndPassword(auth, email, password);
      return { success: true, user: userCredential.user };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const logout = async () => {
    try {
      await signOut(auth);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const value = {
    user,
    login,
    register,
    logout,
    loading
  };

  return (
    <AuthContext.Provider value={value}>
      {!loading && children}
    </AuthContext.Provider>
  );
};
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert,
  KeyboardAvoidingView,
  Platform,
  ScrollView
} from 'react-native';
import { useAuth } from '../utils/AuthContext';

const LoginScreen = ({ navigation }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isLogin, setIsLogin] = useState(true);
  const [loading, setLoading] = useState(false);
  
  const { login, register } = useAuth();

  const handleAuth = async () => {
    if (!email || !password) {
      Alert.alert('Error', 'Por favor completa todos los campos');
      return;
    }

    setLoading(true);
    let result;
    
    if (isLogin) {
      result = await login(email, password);
    } else {
      result = await register(email, password);
    }

    setLoading(false);
    
    if (result.success) {
      navigation.replace('Main');
    } else {
      Alert.alert('Error', result.error);
    }
  };

  return (
    <KeyboardAvoidingView 
      style={styles.container}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <ScrollView contentContainerStyle={styles.scrollContainer}>
        <View style={styles.header}>
          <Text style={styles.logo}>AhorraYa+</Text>
          <Text style={styles.subtitle}>
            {isLogin ? 'Inicia sesión' : 'Crea tu cuenta'}
          </Text>
        </View>

        <View style={styles.form}>
          <TextInput
            style={styles.input}
            placeholder="Email"
            value={email}
            onChangeText={setEmail}
            autoCapitalize="none"
            keyboardType="email-address"
          />
          
          <TextInput
            style={styles.input}
            placeholder="Contraseña"
            value={password}
            onChangeText={setPassword}
            secureTextEntry
          />

          <TouchableOpacity 
            style={[styles.button, loading && styles.buttonDisabled]}
            onPress={handleAuth}
            disabled={loading}
          >
            <Text style={styles.buttonText}>
              {loading ? 'Cargando...' : (isLogin ? 'Iniciar Sesión' : 'Registrarse')}
            </Text>
          </TouchableOpacity>

          <TouchableOpacity 
            style={styles.switchButton}
            onPress={() => setIsLogin(!isLogin)}
          >
            <Text style={styles.switchText}>
              {isLogin ? '¿No tienes cuenta? Regístrate' : '¿Ya tienes cuenta? Inicia sesión'}
            </Text>
          </TouchableOpacity>
        </View>
      </ScrollView>
    </KeyboardAvoidingView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  scrollContainer: {
    flexGrow: 1,
    justifyContent: 'center',
    padding: 20,
  },
  header: {
    alignItems: 'center',
    marginBottom: 50,
  },
  logo: {
    fontSize: 36,
    fontWeight: 'bold',
    color: '#4e54c8',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 18,
    color: '#666',
  },
  form: {
    width: '100%',
  },
  input: {
    backgroundColor: '#f5f5f5',
    padding: 15,
    borderRadius: 10,
    marginBottom: 15,
    fontSize: 16,
  },
  button: {
    backgroundColor: '#4e54c8',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginBottom: 15,
  },
  buttonDisabled: {
    opacity: 0.6,
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
  switchButton: {
    alignItems: 'center',
    padding: 10,
  },
  switchText: {
    color: '#4e54c8',
    fontSize: 14,
  },
});

export default LoginScreen;
import React, { useState, useEffect } from 'react';
import {
  View,
  StyleSheet,
  Alert,
  Text,
  TouchableOpacity,
  FlatList
} from 'react-native';
import MapView, { Marker } from 'react-native-maps';
import * as Location from 'expo-location';
import { collection, onSnapshot, query, where } from 'firebase/firestore';
import { db } from '../config/firebase';

const MapScreen = ({ navigation }) => {
  const [location, setLocation] = useState(null);
  const [offers, setOffers] = useState([]);
  const [errorMsg, setErrorMsg] = useState(null);

  useEffect(() => {
    (async () => {
      let { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') {
        setErrorMsg('Permiso de ubicación denegado');
        return;
      }

      let currentLocation = await Location.getCurrentPositionAsync({});
      setLocation(currentLocation);
    })();
  }, []);

  useEffect(() => {
    const q = query(collection(db, 'offers'), where('active', '==', true));
    
    const unsubscribe = onSnapshot(q, (snapshot) => {
      const offersData = [];
      snapshot.forEach((doc) => {
        offersData.push({ id: doc.id, ...doc.data() });
      });
      setOffers(offersData);
    });

    return () => unsubscribe();
  }, []);

  const renderOfferItem = ({ item }) => (
    <TouchableOpacity 
      style={styles.offerCard}
      onPress={() => navigation.navigate('OfferDetail', { offer: item })}
    >
      <Text style={styles.offerTitle}>{item.title}</Text>
      <Text style={styles.offerDiscount}>{item.discount}</Text>
      <Text style={styles.offerBusiness}>{item.businessName}</Text>
      <Text style={styles.offerDistance}>A 1.2 km</Text>
    </TouchableOpacity>
  );

  if (errorMsg) {
    return (
      <View style={styles.container}>
        <Text style={styles.errorText}>{errorMsg}</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      {location && (
        <MapView
          style={styles.map}
          initialRegion={{
            latitude: location.coords.latitude,
            longitude: location.coords.longitude,
            latitudeDelta: 0.0922,
            longitudeDelta: 0.0421,
          }}
          showsUserLocation={true}
        >
          {offers.map((offer) => (
            <Marker
              key={offer.id}
              coordinate={{
                latitude: offer.location?.latitude || location.coords.latitude + (Math.random() - 0.5) * 0.01,
                longitude: offer.location?.longitude || location.coords.longitude + (Math.random() - 0.5) * 0.01,
              }}
              title={offer.title}
              description={offer.discount}
              onCalloutPress={() => navigation.navigate('OfferDetail', { offer })}
            />
          ))}
        </MapView>
      )}
      
      <View style={styles.offersList}>
        <Text style={styles.sectionTitle}>Ofertas Cercanas</Text>
        <FlatList
          data={offers}
          renderItem={renderOfferItem}
          keyExtractor={item => item.id}
          horizontal
          showsHorizontalScrollIndicator={false}
        />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  map: {
    width: '100%',
    height: '70%',
  },
  offersList: {
    flex: 1,
    padding: 15,
    backgroundColor: '#fff',
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  offerCard: {
    backgroundColor: '#f8f9fa',
    padding: 15,
    borderRadius: 10,
    marginRight: 10,
    width: 200,
    borderLeftWidth: 4,
    borderLeftColor: '#4e54c8',
  },
  offerTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  offerDiscount: {
    fontSize: 14,
    color: '#e74c3c',
    fontWeight: 'bold',
    marginBottom: 5,
  },
  offerBusiness: {
    fontSize: 12,
    color: '#666',
    marginBottom: 5,
  },
  offerDistance: {
    fontSize: 11,
    color: '#999',
  },
  errorText: {
    fontSize: 16,
    color: 'red',
    textAlign: 'center',
    marginTop: 50,
  },
});

export default MapScreen;
import React, { useState } from 'react';
import {
  View,
  Text,
  StyleSheet,
  Image,
  TouchableOpacity,
  ScrollView,
  Alert,
  Share
} from 'react-native';
import QRCode from 'react-native-qrcode-svg';
import { doc, updateDoc, arrayUnion, getDoc } from 'firebase/firestore';
import { auth, db } from '../config/firebase';
import { useAuth } from '../utils/AuthContext';

const OfferDetailScreen = ({ route, navigation }) => {
  const { offer } = route.params;
  const [couponAcquired, setCouponAcquired] = useState(false);
  const [couponCode, setCouponCode] = useState('');
  const { user } = useAuth();

  const generateCouponCode = () => {
    return `AY${Date.now()}${Math.random().toString(36).substr(2, 5)}`.toUpperCase();
  };

  const acquireCoupon = async () => {
    try {
      if (!user) {
        Alert.alert('Error', 'Debes iniciar sesión para obtener cupones');
        return;
      }

      const code = generateCouponCode();
      const couponData = {
        id: Date.now().toString(),
        offerId: offer.id,
        offerTitle: offer.title,
        offerDiscount: offer.discount,
        businessName: offer.businessName,
        acquiredAt: new Date(),
        used: false,
        code: code,
        validUntil: offer.validUntil || new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
      };

      // Guardar en Firebase
      const userRef = doc(db, 'users', user.uid);
      const userDoc = await getDoc(userRef);
      
      if (userDoc.exists()) {
        await updateDoc(userRef, {
          coupons: arrayUnion(couponData)
        });
      } else {
        await updateDoc(userRef, {
          coupons: [couponData],
          email: user.email,
          createdAt: new Date()
        });
      }

      setCouponCode(code);
      setCouponAcquired(true);
      
      Alert.alert('¡Éxito!', 'Cupón obtenido correctamente');
    } catch (error) {
      console.error('Error al adquirir cupón:', error);
      Alert.alert('Error', 'No se pudo obtener el cupón');
    }
  };

  const shareOffer = async () => {
    try {
      await Share.share({
        message: `¡Mira esta oferta en AhorraYa+! ${offer.title} - ${offer.discount}`,
        url: 'https://ahorraya.com', // Reemplazar con tu URL real
        title: offer.title
      });
    } catch (error) {
      console.error('Error al compartir:', error);
    }
  };

  return (
    <ScrollView style={styles.container}>
      <Image 
        source={{ uri: offer.image || 'https://via.placeholder.com/300x200?text=Oferta' }}
        style={styles.image}
        defaultSource={require('../../assets/placeholder.jpg')} // Asegúrate de tener esta imagen
      />
      
      <View style={styles.content}>
        <Text style={styles.title}>{offer.title}</Text>
        <Text style={styles.business}>{offer.businessName}</Text>
        <Text style={styles.discount}>{offer.discount}</Text>
        
        <Text style={styles.description}>
          {offer.description || 'Oferta especial para usuarios de AhorraYa+'}
        </Text>
        
        <View style={styles.details}>
          <Text style={styles.detailText}>
            📍 {offer.address || 'Ubicación no especificada'}
          </Text>
          <Text style={styles.detailText}>
            ⏰ Válido hasta: {offer.validUntil ? new Date(offer.validUntil).toLocaleDateString() : '31/12/2023'}
          </Text>
          <Text style={styles.detailText}>
            🏷️ Categoría: {offer.category || 'General'}
          </Text>
        </View>

        {couponAcquired ? (
          <View style={styles.couponSection}>
            <Text style={styles.congrats}>¡Cupón Obtenido!</Text>
            <Text style={styles.couponCode}>Código: {couponCode}</Text>
            
            <View style={styles.qrContainer}>
              <QRCode
                value={couponCode}
                size={200}
              />
            </View>
            
            <Text style={styles.instructions}>
              Muestra este código QR o el código numérico en el establecimiento
            </Text>
            
            <TouchableOpacity style={styles.secondaryButton} onPress={shareOffer}>
              <Text style={styles.secondaryButtonText}>Compartir Oferta</Text>
            </TouchableOpacity>
          </View>
        ) : (
          <View style={styles.actionSection}>
            <TouchableOpacity style={styles.primaryButton} onPress={acquireCoupon}>
              <Text style={styles.primaryButtonText}>Obtener Cupón Gratis</Text>
            </TouchableOpacity>
            
            <TouchableOpacity style={styles.secondaryButton} onPress={shareOffer}>
              <Text style={styles.secondaryButtonText}>Compartir con Amigos</Text>
            </TouchableOpacity>
          </View>
        )}
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  image: {
    width: '100%',
    height: 250,
  },
  content: {
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 5,
    color: '#333',
  },
  business: {
    fontSize: 16,
    color: '#666',
    marginBottom: 10,
  },
  discount: {
    fontSize: 20,
    color: '#e74c3c',
    fontWeight: 'bold',
    marginBottom: 15,
  },
  description: {
    fontSize: 16,
    lineHeight: 22,
    color: '#555',
    marginBottom: 20,
  },
  details: {
    backgroundColor: '#f8f9fa',
    padding: 15,
    borderRadius: 10,
    marginBottom: 20,
  },
  detailText: {
    fontSize: 14,
    marginBottom: 5,
    color: '#666',
  },
  actionSection: {
    marginTop: 20,
  },
  primaryButton: {
    backgroundColor: '#4e54c8',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginBottom: 10,
  },
  primaryButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
  secondaryButton: {
    backgroundColor: 'transparent',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    borderWidth: 1,
    borderColor: '#4e54c8',
  },
  secondaryButtonText: {
    color: '#4e54c8',
    fontSize: 16,
    fontWeight: 'bold',
  },
  couponSection: {
    marginTop: 20,
    alignItems: 'center',
  },
  congrats: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#27ae60',
    marginBottom: 10,
  },
  couponCode: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 20,
  },
  qrContainer: {
    padding: 20,
    backgroundColor: '#fff',
    borderRadius: 10,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  instructions: {
    fontSize: 14,
    color: '#666',
    textAlign: 'center',
    marginBottom: 20,
  },
});

export default OfferDetailScreen;
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  FlatList,
  TouchableOpacity,
  Alert
} from 'react-native';
import { doc, onSnapshot, updateDoc } from 'firebase/firestore';
import { db } from '../config/firebase';
import { useAuth } from '../utils/AuthContext';

const CouponsScreen = () => {
  const [coupons, setCoupons] = useState([]);
  const { user } = useAuth();

  useEffect(() => {
    if (!user) return;

    const userRef = doc(db, 'users', user.uid);
    
    const unsubscribe = onSnapshot(userRef, (docSnap) => {
      if (docSnap.exists()) {
        const userData = docSnap.data();
        setCoupons(userData.coupons || []);
      }
    });

    return () => unsubscribe();
  }, [user]);

  const markAsUsed = async (couponId) => {
    try {
      const userRef = doc(db, 'users', user.uid);
      const updatedCoupons = coupons.map(coupon => 
        coupon.id === couponId ? { ...coupon, used: true } : coupon
      );
      
      await updateDoc(userRef, {
        coupons: updatedCoupons
      });
      
      Alert.alert('¡Listo!', 'Cupón marcado como usado');
    } catch (error) {
      console.error('Error al marcar cupón:', error);
      Alert.alert('Error', 'No se pudo marcar el cupón');
    }
  };

  const renderCouponItem = ({ item }) => (
    <View style={[styles.couponCard, item.used && styles.usedCoupon]}>
      <View style={styles.couponHeader}>
        <Text style={styles.couponTitle}>{item.offerTitle}</Text>
        <Text style={styles.couponDiscount}>{item.offerDiscount}</Text>
      </View>
      
      <Text style={styles.businessName}>{item.businessName}</Text>
      <Text style={styles.couponCode}>Código: {item.code}</Text>
      <Text style={styles.validUntil}>
        Válido hasta: {new Date(item.validUntil).toLocaleDateString()}
      </Text>
      
      {!item.used && (
        <TouchableOpacity 
          style={styles.useButton}
          onPress={() => markAsUsed(item.id)}
        >
          <Text style={styles.useButtonText}>Marcar como Usado</Text>
        </TouchableOpacity>
      )}
      
      {item.used && (
        <Text style={styles.usedText}>✅ Ya usado</Text>
      )}
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Mis Cupones</Text>
      
      {coupons.length === 0 ? (
        <View style={styles.emptyState}>
          <Text style={styles.emptyText}>No tienes cupones aún</Text>
          <Text style={styles.emptySubtext}>
            Visita el mapa para descubrir ofertas y obtener cupones
          </Text>
        </View>
      ) : (
        <FlatList
          data={coupons}
          renderItem={renderCouponItem}
          keyExtractor={item => item.id}
          showsVerticalScrollIndicator={false}
        />
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    padding: 15,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    color: '#333',
  },
  couponCard: {
    backgroundColor: '#f8f9fa',
    padding: 15,
    borderRadius: 10,
    marginBottom: 15,
    borderLeftWidth: 4,
    borderLeftColor: '#4e54c8',
  },
  usedCoupon: {
    opacity: 0.7,
    borderLeftColor: '#95a5a6',
  },
  couponHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'flex-start',
    marginBottom: 10,
  },
  couponTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    flex: 1,
    marginRight: 10,
  },
  couponDiscount: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#e74c3c',
  },
  businessName: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5,
  },
  couponCode: {
    fontSize: 12,
    fontFamily: 'monospace',
    backgroundColor: '#e9ecef',
    padding: 5,
    borderRadius: 5,
    marginBottom: 5,
  },
  validUntil: {
    fontSize: 12,
    color: '#999',
    marginBottom: 10,
  },
  useButton: {
    backgroundColor: '#4e54c8',
    padding: 10,
    borderRadius: 5,
    alignItems: 'center',
  },
  useButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: 'bold',
  },
  usedText: {
    color: '#27ae60',
    fontSize: 14,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  emptyState: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  emptyText: {
    fontSize: 18,
    color: '#666',
    marginBottom: 10,
  },
  emptySubtext: {
    fontSize: 14,
    color: '#999',
    textAlign: 'center',
  },
});

export default CouponsScreen;
import React from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Alert,
  ScrollView
} from 'react-native';
import { useAuth } from '../utils/AuthContext';

const ProfileScreen = () => {
  const { user, logout } = useAuth();

  const handleLogout = async () => {
    Alert.alert(
      'Cerrar Sesión',
      '¿Estás seguro de que quieres cerrar sesión?',
      [
        { text: 'Cancelar', style: 'cancel' },
        { 
          text: 'Cerrar Sesión', 
          style: 'destructive',
          onPress: async () => {
            const result = await logout();
            if (!result.success) {
              Alert.alert('Error', 'No se pudo cerrar sesión');
            }
          }
        },
      ]
    );
  };

  const stats = [
    { label: 'Cupones Obtenidos', value: '12' },
    { label: 'Dinero Ahorrado', value: '$156.50' },
    { label: 'Ofertas Visitadas', value: '8' },
  ];

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <View style={styles.avatar}>
          <Text style={styles.avatarText}>
            {user?.email?.charAt(0).toUpperCase()}
          </Text>
        </View>
        <Text style={styles.email}>{user?.email}</Text>
        <Text style={styles.memberSince}>Miembro desde Enero 2023</Text>
      </View>

      <View style={styles.statsContainer}>
        <Text style={styles.sectionTitle}>Mi Actividad</Text>
        <View style={styles.statsGrid}>
          {stats.map((stat, index) => (
            <View key={index} style={styles.statCard}>
              <Text style={styles.statValue}>{stat.value}</Text>
              <Text style={styles.statLabel}>{stat.label}</Text>
            </View>
          ))}
        </View>
      </View>

      <View style={styles.menuSection}>
        <Text style={styles.sectionTitle}>Configuración</Text>
        
        <TouchableOpacity style={styles.menuItem}>
          <Text style={styles.menuText}>Notificaciones</Text>
          <Text style={styles.menuArrow}>›</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.menuItem}>
          <Text style={styles.menuText}>Privacidad</Text>
          <Text style={styles.menuArrow}>›</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.menuItem}>
          <Text style={styles.menuText}>Ayuda y Soporte</Text>
          <Text style={styles.menuArrow}>›</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.menuItem}>
          <Text style={styles.menuText}>Términos y Condiciones</Text>
          <Text style={styles.menuArrow}>›</Text>
        </TouchableOpacity>
      </View>

      <TouchableOpacity style={styles.logoutButton} onPress={handleLogout}>
        <Text style={styles.logoutButtonText}>Cerrar Sesión</Text>
      </TouchableOpacity>

      <Text style={styles.version}>AhorraYa+ v1.0.0</Text>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  header: {
    alignItems: 'center',
    padding: 30,
    backgroundColor: '#f8f9fa',
  },
  avatar: {
    width: 80,
    height: 80,
    borderRadius: 40,
    backgroundColor: '#4e54c8',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 15,
  },
  avatarText: {
    color: '#fff',
    fontSize: 30,
    fontWeight: 'bold',
  },
  email: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5,
  },
  memberSince: {
    fontSize: 14,
    color: '#666',
  },
  statsContainer: {
    padding: 20,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  statsGrid: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  statCard: {
    flex: 1,
    backgroundColor: '#f8f9fa',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginHorizontal: 5,
  },
  statValue: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#4e54c8',
    marginBottom: 5,
  },
  statLabel: {
    fontSize: 12,
    color: '#666',
    textAlign: 'center',
  },
  menuSection: {
    padding: 20,
  },
  menuItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0',
  },
  menuText: {
    fontSize: 16,
    color: '#333',
  },
  menuArrow: {
    fontSize: 20,
    color: '#999',
  },
  logoutButton: {
    margin: 20,
    padding: 15,
    backgroundColor: '#e74c3c',
    borderRadius: 10,
    alignItems: 'center',
  },
  logoutButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
  version: {
    textAlign: 'center',
    color: '#999',
    fontSize: 12,
    marginBottom: 20,
  },
});

export default ProfileScreen;

# Si prefieres usar Git:
1. Crea un repositorio en GitHub
2. Sube tu archivo index.html
3. Conecta Netlify a tu repositorio
4. Despliegue automático
