# Moonberry Ecosystem

> **COMMERCIAL PROJECT DISCLAIMER**
>
> This project, **Moonberry**, is a live commercial product developed under **Vectorium Technology**. The source code is **private and proprietary** protected by NDA.
>
> This repository serves as a **technical showcase and architectural overview** for portfolio purposes. It demonstrates engineering methodologies, tech stack proficiency, and system design capabilities without revealing the full source code.

---

## Project Overview

**Moonberry** is a comprehensive, full-stack digital ecosystem designed for the modern premium coffee retail industry. It seamlessly bridges the gap between digital customer engagement and physical store operations.

The ecosystem consists of three synchronized pillars:
1.  **Customer Loyalty App (Mobile):** A premium React Native implementation for end-users to order ahead, manage digital wallets, and engage with gamified loyalty features.
2.  **Cashier Operations App (Tablet/Mobile):** A robust Point-of-Sale (POS) interface for store staff to accept orders, manage queue status in real-time, and process payments.
3.  **Central Intelligence Core (Backend):** A monolithic Architecture with modular design (Django) that orchestrates inventory, payments, user tiers, and real-time socket communications.

---

## Tech Stack

### Frontend (Mobile Ecosystem)
*   **Framework:** React Native (Expo SDK 54) / TypeScript
*   **State Management:** Zustand (Client state), TanStack Query (Server state)
*   **Navigation:** React Navigation v7 (Native Stack & Bottom Tabs)
*   **UI/UX:** Reanimated 2, Expo Haptics, Expo Linear Gradient, Blur Views
*   **Hardware Integration:** Camera (QR Scanning), Sensors, Secure Store
*   **Gamification:** Custom SVG-based Wheel of Fortune, Interactive Animations

### Backend (API & Core)
*   **Framework:** Python 3.12, Django 5.x, Django REST Framework (DRF)
*   **Real-Time:** Django Channels, Redis, Daphne (WebSockets for Order Status)
*   **Async Processing:** Celery & Redis (Task Queue for Notifications/Campaigns)
*   **Database:** PostgreSQL
*   **Admin Interface:** Heavily customized `django-jazzmin` & `django-admin-interface`

### Integrations & Services
*   **Payments:** PayTR Infrastructure Integration
*   **Notifications:** Verified SMS (NetGSM), Push Notifications
*   **Analytics:** Custom internal analytics + Exportable Reports (Pandas/OpenPyXL)

---

## System Architecture

The project follows a **Modular Monolith** architecture, ensuring code maintainability while keeping infrastructure simple and robust.

### Key Architectural Patterns:
*   **Event-Driven Communication:** Utilizing WebSockets (Django Channels) to push order status updates from Cashier -> Backend -> Customer App instantly.
*   **Service-Repository Pattern:** Business logic is decoupled from Views/API layers, residing in dedicated `services/` modules.
*   **Gamification Engine:** A dedicated logic layer handling probability, user tiers, and reward distribution.
*   **Role-Based Access Control (RBAC):** Distinct authentication flows for Customers vs. Staff (Cashiers), ensuring strict data isolation.

---

## Key Features & Engineering Highlights

### 1. Real-Time Order Management
*   **Syncing:** Syncing order status between the kitchen/barista (Cashier App) and the customer waiting outside (Loyalty App) with zero latency.
*   **Implementation:** Implemented a WebSocket layer. When a cashier updates an order to "Ready", a socket event triggers a haptic notification on the customer's device immediately.

### 2. Digital Wallet & Payment Security
*   **Transactions:** Handling stored value (Coffee Credits) and External Payments secure.
*   **Implementation:** Built a double-ledger system in `apps/wallet` to track credit transactions. Integrated PayTR with reliable callback handling for credit card top-ups.

### 3. Smart Gamification & Loyalty
*   **Feature:** "Spin & Win" functionality.
*   **Engineering:** Logic is server-side authoritative to prevent tampering. The frontend uses `react-native-reanimated` for smooth physics-based spinning visuals that align perfectly with the backend's determined result.

### 4. Custom Admin & Analytics
*   **Insights:** Business owners need detailed insights without technical SQL knowledge.
*   **Engineering:** Extended the Django Admin with custom views, charts, and Excel export capabilities (`pandas` integration) for sales reports, stock levels, and user retention metrics.

---

## Key Code Highlights

### Backend: Atomic State Transitions
This snippet from `OrderStatusManager` demonstrates robust state management using Django atomic transactions to ensure data consistency during order processing. It handles validation, history logging, and notifications in a single transaction.

```python
class OrderStatusManager:
    """
    Central class for order status management.
    Controls state transitions and performs automatic updates.
    """
    
    @classmethod
    @transaction.atomic
    def update_status(cls, order, new_status, user=None, description=None, auto=False):
        try:
            # Validate state transition
            cls.validate_transition(order, new_status, user)
            
            old_status = order.status
            
            # Apply new status
            order.status = new_status
            order.updated_at = timezone.now()
            order.save()
            
            # Log to History
            OrderStatusHistory.objects.create(
                order=order,
                status=new_status,
                description=description or f"Status updated: '{old_status}' -> '{new_status}'",
                is_automatic=auto
            )
            
            # Schedule async follow-up tasks
            cls._schedule_auto_transitions(order)
            
            # Trigger real-time notifications (Push/Socket)
            cls._send_status_notification(order, old_status, new_status)
            
            return True
            
        except ValidationError as e:
            logger.warning(f"Order {order.id} update error: {str(e)}")
            raise
```

### Frontend: OTA Updates Hook
A custom hook implementation for handling Over-The-Air (OTA) updates using Expo Updates, allowing critical hotfixes to be deployed without app store reviews.

```javascript
export const useUpdates = () => {
    useEffect(() => {
        if (__DEV__) return;

        const checkUpdates = async () => {
            // Allow app initialization
            await new Promise(resolve => setTimeout(resolve, 3000));

            try {
                const update = await Updates.checkForUpdateAsync();
                if (update.isAvailable) {
                    await Updates.fetchUpdateAsync();

                    Alert.alert(
                        'Update Ready',
                        'A new version has been downloaded. Restart now to apply?',
                        [
                            { text: 'Later', style: 'cancel' },
                            {
                                text: 'Restart Now',
                                onPress: async () => {
                                    await Updates.reloadAsync();
                                }
                            },
                        ]
                    );
                }
            } catch (error) {
                console.log('Update check skipped:', error.message);
            }
        };

        checkUpdates();
    }, []);
};
```

---

## Visuals

| Customer Home | Spin Wheel | Cashier Dashboard |
|:---:|:---:|:---:|
| ![Customer Home](./assets/customer_home_placeholder.png) | ![Spin Wheel](./assets/spin_wheel_placeholder.png) | ![Cashier Panel](./assets/cashier_placeholder.png) |

---

## Team

**Developed by the Engineering Team at Vectorium Technology**

* **Mehmet Furkan Güneş** - Co-Founder & Full Stack Engineer
    * *Focus:* Mobile Architecture (React Native), UI/UX Design, Client-Side Logic.
* **Nihal Kemer** - Co-Founder & Full Stack Engineer
    * *Focus:* System Architecture, API Development (Django), Database Optimization.
