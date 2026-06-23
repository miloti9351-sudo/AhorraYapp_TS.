{
  "name": "ahorrayaapp",
  "version": "1.0.0",
  "main": "node_modules/expo/AppEntry.js",
  "scripts": {
    "start" "expo build",
    "start" "expo star --web",
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
