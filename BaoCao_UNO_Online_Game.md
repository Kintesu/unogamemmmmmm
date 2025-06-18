# BÁO CÁO DỰ ÁN UNO ONLINE GAME

## THÔNG TIN CHUNG

**Tên dự án:** UNO Online Game  
**Ngôn ngữ:** TypeScript, JavaScript  
**Framework:** React 18, Node.js, Express  
**Cơ sở dữ liệu:** In-memory storage  
**Giao tiếp real-time:** Socket.IO  
**Ngày hoàn thành:** 2024  

---

## 1. TỔNG QUAN DỰ ÁN

### 1.1 Mục tiêu dự án
Phát triển một ứng dụng web game UNO trực tuyến cho phép nhiều người chơi cùng tham gia từ các thiết bị khác nhau qua internet. Game hỗ trợ chơi real-time với đầy đủ tính năng của UNO truyền thống cùng các lá bài mở rộng.

### 1.2 Đối tượng sử dụng
- Người chơi game UNO muốn chơi trực tuyến với bạn bè
- Các nhóm bạn muốn giải trí từ xa
- Người dùng muốn trải nghiệm UNO với các tính năng mở rộng

### 1.3 Phạm vi dự án
- Ứng dụng web responsive hoạt động trên mọi thiết bị
- Hỗ trợ multiplayer real-time (2-8 người chơi)
- Hệ thống phòng chơi với mật khẩu tùy chọn
- Chat và sticker trong game
- AI players cho chế độ single-player

---

## 2. PHÂN TÍCH YÊU CẦU

### 2.1 Yêu cầu chức năng

#### 2.1.1 Hệ thống phòng chơi
- **Tạo phòng mới:** Người dùng có thể tạo phòng với tên tùy chỉnh, số người chơi tối đa (2-8), và mật khẩu tùy chọn
- **Tham gia phòng:** Duyệt danh sách phòng hoặc nhập trực tiếp Room ID
- **Quản lý phòng:** Host có thể kick người chơi, bắt đầu game
- **Trạng thái sẵn sàng:** Người chơi có thể toggle trạng thái ready

#### 2.1.2 Gameplay UNO
- **Lá bài cơ bản:** Number cards (0-9), Skip, Reverse, Draw 2, Wild, Wild Draw 4
- **Lá bài mở rộng:** Swap Hands, Draw Minus 2, Shuffle My Hand, Block All
- **Luật chơi:** Ghép theo màu, số, hoặc ký hiệu
- **Gọi UNO:** Bắt buộc khi còn 1 lá bài
- **Stacking system:** Có thể cộng +2 với +2/+4, +4 chỉ cộng với +4
- **Elimination:** Người chơi có 35+ bài sẽ bị loại

#### 2.1.3 Tính năng real-time
- **Đồng bộ game state:** Mọi hành động được đồng bộ ngay lập tức
- **Chat system:** Tin nhắn text và sticker
- **Live updates:** Cập nhật trạng thái phòng, người chơi real-time

#### 2.1.4 Tính năng bổ sung
- **Unlimited cards:** Bộ bài tự động xáo trộn khi hết
- **Responsive design:** Tương thích mọi thiết bị
- **Connection handling:** Tự động reconnect khi mất kết nối

### 2.2 Yêu cầu phi chức năng

#### 2.2.1 Hiệu năng
- Thời gian phản hồi < 100ms cho các hành động game
- Hỗ trợ đồng thời tối đa 100 phòng chơi
- Tự động cleanup phòng không hoạt động sau 30 phút

#### 2.2.2 Bảo mật
- Validation tất cả input từ client
- Chống cheating bằng server-side validation
- CORS protection cho API endpoints

#### 2.2.3 Khả năng mở rộng
- Architecture modular dễ bảo trì
- Code được tổ chức theo component
- Dễ dàng thêm tính năng mới

---

## 3. THIẾT KẾ HỆ THỐNG

### 3.1 Kiến trúc tổng thể

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Storage       │
│   (React)       │◄──►│   (Node.js)     │◄──►│   (In-memory)   │
│                 │    │                 │    │                 │
│ - UI Components │    │ - Socket.IO     │    │ - Rooms         │
│ - Game Logic    │    │ - Game Engine   │    │ - Players       │
│ - State Mgmt    │    │ - Room Mgmt     │    │ - Game States   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 3.2 Kiến trúc Frontend

#### 3.2.1 Cấu trúc thư mục
```
src/
├── components/          # UI Components
│   ├── Chat/           # Chat system
│   ├── RoomSystem/     # Room management
│   ├── Card.tsx        # Card component
│   ├── GameBoard.tsx   # Main game area
│   ├── GameStatus.tsx  # Game status display
│   └── PlayerHand.tsx  # Player hand display
├── hooks/              # Custom React hooks
│   ├── useGameState.ts # Local game state
│   ├── useRoomSystem.ts# Room management
│   ├── useChatSystem.ts# Chat functionality
│   └── useAI.ts        # AI player logic
├── services/           # External services
│   └── SocketService.ts# WebSocket communication
├── types/              # TypeScript definitions
│   ├── Card.ts         # Game types
│   ├── Room.ts         # Room types
│   └── Chat.ts         # Chat types
├── utils/              # Utility functions
│   └── cardUtils.ts    # Card game logic
└── data/               # Static data
    └── stickers.ts     # Sticker definitions
```

#### 3.2.2 State Management
- **Local State:** React useState cho UI state
- **Game State:** Custom hooks cho game logic
- **Global State:** Socket.IO events cho real-time sync

### 3.3 Kiến trúc Backend

#### 3.3.1 Cấu trúc server
```
server/
└── index.js           # Main server file
    ├── Express setup  # REST API endpoints
    ├── Socket.IO      # WebSocket handling
    ├── Room management# Phòng chơi
    ├── Game engine    # Logic game
    └── Chat system    # Hệ thống chat
```

#### 3.3.2 Data Models
```javascript
// Room Model
{
  id: string,
  name: string,
  hostId: string,
  players: Player[],
  maxPlayers: number,
  hasPassword: boolean,
  status: 'waiting' | 'playing',
  gameState: GameState | null
}

// Player Model
{
  id: string,
  name: string,
  socketId: string,
  isHost: boolean,
  isReady: boolean,
  cards: Card[]
}

// Game State Model
{
  players: Player[],
  currentPlayerIndex: number,
  direction: 'clockwise' | 'counterclockwise',
  topCard: Card,
  drawPile: Card[],
  wildColor?: CardColor,
  stackingType: 'none' | 'draw-two' | 'wild-draw-four',
  stackedDrawCount: number
}
```

---

## 4. CÔNG NGHỆ SỬ DỤNG

### 4.1 Frontend Technologies

| Công nghệ | Phiên bản | Mục đích |
|-----------|-----------|----------|
| React | 18.3.1 | UI Framework |
| TypeScript | 5.5.3 | Type safety |
| Tailwind CSS | 3.4.1 | Styling |
| Vite | 5.4.2 | Build tool |
| Socket.IO Client | 4.7.4 | Real-time communication |
| Lucide React | 0.344.0 | Icons |

### 4.2 Backend Technologies

| Công nghệ | Phiên bản | Mục đích |
|-----------|-----------|----------|
| Node.js | Latest | Runtime environment |
| Express | 4.18.2 | Web framework |
| Socket.IO | 4.7.4 | WebSocket server |
| CORS | 2.8.5 | Cross-origin requests |
| UUID | 9.0.1 | Unique ID generation |

### 4.3 Development Tools

| Tool | Mục đích |
|------|----------|
| ESLint | Code linting |
| Prettier | Code formatting |
| Concurrently | Run multiple scripts |
| PostCSS | CSS processing |
| Autoprefixer | CSS vendor prefixes |

---

## 5. TÍNH NĂNG CHI TIẾT

### 5.1 Hệ thống phòng chơi

#### 5.1.1 Tạo phòng
- **Input validation:** Kiểm tra tên phòng, tên người chơi
- **Room ID generation:** Tạo ID ngẫu nhiên 6 ký tự
- **Password protection:** Mã hóa mật khẩu phòng
- **Host privileges:** Quyền quản lý phòng

#### 5.1.2 Tham gia phòng
- **Room browser:** Hiển thị danh sách phòng công khai
- **Direct join:** Nhập trực tiếp Room ID
- **Password verification:** Xác thực mật khẩu
- **Capacity check:** Kiểm tra số lượng người chơi

#### 5.1.3 Quản lý phòng
- **Player management:** Kick, ready status
- **Game control:** Start, end, restart game
- **Real-time updates:** Sync trạng thái phòng

### 5.2 Game Engine

#### 5.2.1 Card System
```typescript
// Card Types
type CardType = 'number' | 'skip' | 'reverse' | 'draw-two' | 
                'wild' | 'wild-draw-four' | 'swap-hands' | 
                'draw-minus-two' | 'shuffle-my-hand' | 'block-all';

// Deck Creation
- 108 lá bài chuẩn UNO
- 32 lá bài mở rộng
- Tổng cộng 140 lá bài
```

#### 5.2.2 Game Rules Implementation
```typescript
// Card Validation
function canPlayCard(card: Card, topCard: Card, wildColor?: CardColor): boolean {
  // Wild cards always playable
  if (card.type === 'wild' || card.type === 'wild-draw-four') return true;
  
  // Wild color priority
  if (wildColor) return card.color === wildColor;
  
  // Standard UNO rules
  return card.color === topCard.color || 
         card.type === topCard.type || 
         (card.type === 'number' && topCard.type === 'number' && 
          card.value === topCard.value);
}
```

#### 5.2.3 Stacking System
- **+2 Stacking:** +2 có thể cộng với +2 hoặc +4
- **+4 Stacking:** +4 chỉ có thể cộng với +4
- **Stack Resolution:** Người không thể cộng phải rút tất cả

#### 5.2.4 Special Cards Effects
```typescript
// Swap Hands: Đổi bài với người chơi khác
// Draw Minus 2: Bỏ 2 lá hoặc rút 2 lá
// Shuffle My Hand: Bỏ tất cả và rút lại
// Block All: Chỉ cho phép đánh lá số
```

### 5.3 Real-time Communication

#### 5.3.1 Socket Events
```typescript
// Client to Server
'create-room', 'join-room', 'leave-room'
'start-game', 'toggle-ready'
'broadcast-card-play', 'broadcast-draw-card'
'send-chat-message'

// Server to Client
'rooms-updated', 'player-joined', 'player-left'
'game-started', 'game-state-update'
'chat-message', 'kicked-from-room'
```

#### 5.3.2 Game State Synchronization
- **Host Authority:** Host quản lý game state chính thức
- **Action Broadcasting:** Mọi hành động được broadcast
- **State Validation:** Server validate mọi action
- **Conflict Resolution:** Host có quyền quyết định cuối cùng

### 5.4 Chat System

#### 5.4.1 Message Types
```typescript
interface ChatMessage {
  id: string;
  playerId: string;
  playerName: string;
  type: 'text' | 'sticker';
  content: string;
  timestamp: Date;
  roomId: string;
}
```

#### 5.4.2 Sticker System
- **4 categories:** Emotions, Reactions, Game, Fun
- **32 stickers total:** Emoji-based với tên mô tả
- **Quick access:** Sticker phổ biến dễ truy cập

---

## 6. GIAO DIỆN NGƯỜI DÙNG

### 6.1 Design System

#### 6.1.1 Color Palette
```css
/* Primary Colors */
--purple-900: #581c87;
--blue-900: #1e3a8a;
--indigo-900: #312e81;

/* Card Colors */
--red: #ef4444;
--blue: #3b82f6;
--green: #22c55e;
--yellow: #eab308;
--wild: #8b5cf6;
```

#### 6.1.2 Typography
- **Font Family:** System fonts (sans-serif)
- **Font Weights:** 400 (normal), 600 (semibold), 700 (bold)
- **Font Sizes:** 12px - 48px responsive scale

#### 6.1.3 Spacing System
- **Base unit:** 8px
- **Scale:** 4px, 8px, 12px, 16px, 24px, 32px, 48px, 64px

### 6.2 Component Design

#### 6.2.1 Card Component
```typescript
// Visual Features
- Gradient backgrounds cho mỗi màu
- Corner symbols (top-left, bottom-right rotated)
- Hover effects và animations
- Size variants: small, medium, large
- Playable state indicators
```

#### 6.2.2 Game Board
```typescript
// Layout
- Center-focused design
- Draw pile (left) và discard pile (right)
- Direction indicator
- Stacking status display
- Color picker modal cho wild cards
```

#### 6.2.3 Player Hand
```typescript
// Features
- Overlapping card layout
- Hover lift effects
- Playable card highlighting
- UNO call button
- Card count display
```

### 6.3 Responsive Design

#### 6.3.1 Breakpoints
```css
/* Mobile First Approach */
sm: 640px   /* Small devices */
md: 768px   /* Medium devices */
lg: 1024px  /* Large devices */
xl: 1280px  /* Extra large devices */
```

#### 6.3.2 Mobile Optimizations
- Touch-friendly button sizes (44px minimum)
- Swipe gestures cho card selection
- Optimized chat panel cho mobile
- Responsive grid layouts

---

## 7. TESTING & QUALITY ASSURANCE

### 7.1 Testing Strategy

#### 7.1.1 Unit Testing
- **Card validation logic:** Test tất cả card rules
- **Game state transitions:** Test state changes
- **Utility functions:** Test helper functions

#### 7.1.2 Integration Testing
- **Socket communication:** Test real-time events
- **Room management:** Test room lifecycle
- **Game flow:** Test complete game scenarios

#### 7.1.3 Manual Testing
- **Cross-browser compatibility:** Chrome, Firefox, Safari, Edge
- **Device testing:** Desktop, tablet, mobile
- **Network conditions:** Slow 3G, WiFi, offline

### 7.2 Performance Optimization

#### 7.2.1 Frontend Optimizations
```typescript
// Code Splitting
const RoomBrowser = lazy(() => import('./components/RoomBrowser'));

// Memoization
const MemoizedCard = React.memo(Card);

// Debouncing
const debouncedSearch = useMemo(() => debounce(search, 300), []);
```

#### 7.2.2 Backend Optimizations
```javascript
// Connection pooling
// Memory management
// Cleanup inactive rooms
setInterval(cleanupRooms, 5 * 60 * 1000);

// Rate limiting
// Input validation
// Error handling
```

---

## 8. DEPLOYMENT & DEVOPS

### 8.1 Development Environment

#### 8.1.1 Local Setup
```bash
# Install dependencies
npm install

# Start development servers
npm run dev:full  # Both frontend and backend

# Individual servers
npm run dev      # Frontend only
npm run server   # Backend only
```

#### 8.1.2 Environment Variables
```env
# Frontend (.env)
VITE_SERVER_URL=http://localhost:3001

# Backend
PORT=3001
NODE_ENV=development
```

### 8.2 Production Deployment

#### 8.2.1 Build Process
```bash
# Frontend build
npm run build

# Backend deployment
npm run server
```

#### 8.2.2 Hosting Options
- **Frontend:** Netlify, Vercel, GitHub Pages
- **Backend:** Heroku, Railway, DigitalOcean
- **Full-stack:** AWS, Google Cloud, Azure

### 8.3 Monitoring & Maintenance

#### 8.3.1 Health Checks
```javascript
// Health endpoint
app.get('/api/health', (req, res) => {
  res.json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    activeRooms: rooms.size,
    connectedPlayers: players.size
  });
});
```

#### 8.3.2 Logging & Analytics
- **Server logs:** Connection events, errors, performance
- **Client analytics:** User interactions, game completion rates
- **Error tracking:** Crash reports, bug tracking

---

## 9. BẢO MẬT & PRIVACY

### 9.1 Security Measures

#### 9.1.1 Input Validation
```typescript
// Server-side validation
function validateRoomData(data: CreateRoomData): boolean {
  return data.name.length <= 50 &&
         data.hostName.length <= 20 &&
         data.maxPlayers >= 2 && data.maxPlayers <= 8;
}
```

#### 9.1.2 Anti-Cheating
- **Server authority:** Tất cả game logic trên server
- **Action validation:** Validate mọi player action
- **State consistency:** Đồng bộ state từ authoritative source

#### 9.1.3 CORS Protection
```javascript
const corsOptions = {
  origin: ["http://localhost:5173", "https://yourdomain.com"],
  methods: ["GET", "POST"],
  credentials: true
};
```

### 9.2 Privacy Protection

#### 9.2.1 Data Handling
- **No persistent storage:** Tất cả data in-memory
- **Auto cleanup:** Rooms tự động xóa sau 30 phút
- **Minimal data collection:** Chỉ lưu thông tin cần thiết

#### 9.2.2 User Privacy
- **No registration required:** Chơi anonymous
- **Temporary sessions:** Không lưu user data
- **Local storage minimal:** Chỉ lưu preferences

---

## 10. HƯỚNG PHÁT TRIỂN TƯƠNG LAI

### 10.1 Tính năng mở rộng

#### 10.1.1 Game Features
- **Tournament mode:** Giải đấu nhiều vòng
- **Custom rules:** Tùy chỉnh luật chơi
- **More card types:** Thêm lá bài đặc biệt
- **Power-ups:** Items đặc biệt trong game

#### 10.1.2 Social Features
- **Friend system:** Kết bạn và invite
- **Leaderboards:** Bảng xếp hạng
- **Achievements:** Hệ thống thành tích
- **Profile customization:** Tùy chỉnh avatar

#### 10.1.3 Technical Improvements
- **Database integration:** PostgreSQL/MongoDB
- **User authentication:** JWT-based auth
- **Mobile app:** React Native version
- **AI improvements:** Smarter AI opponents

### 10.2 Scalability Plans

#### 10.2.1 Infrastructure
- **Microservices:** Tách game engine, room service
- **Load balancing:** Multiple server instances
- **CDN integration:** Static asset delivery
- **Caching layer:** Redis for session management

#### 10.2.2 Performance
- **WebRTC:** Peer-to-peer communication
- **Compression:** Message compression
- **Optimization:** Bundle size reduction
- **Progressive loading:** Lazy load components

---

## 11. KẾT LUẬN

### 11.1 Thành tựu đạt được

#### 11.1.1 Technical Achievements
- ✅ **Real-time multiplayer:** Hoạt động ổn định với WebSocket
- ✅ **Complete UNO implementation:** Đầy đủ luật chơi + mở rộng
- ✅ **Responsive design:** Tương thích mọi thiết bị
- ✅ **Unlimited card system:** Bộ bài không giới hạn
- ✅ **Chat integration:** Hệ thống chat với sticker
- ✅ **Room management:** Quản lý phòng chơi hoàn chỉnh

#### 11.1.2 User Experience
- ✅ **Intuitive interface:** Giao diện dễ sử dụng
- ✅ **Smooth gameplay:** Trải nghiệm mượt mà
- ✅ **Visual feedback:** Hiệu ứng và animations
- ✅ **Error handling:** Xử lý lỗi graceful
- ✅ **Connection resilience:** Tự động reconnect

### 11.2 Lessons Learned

#### 11.2.1 Technical Insights
- **Real-time sync complexity:** Đồng bộ state phức tạp hơn dự kiến
- **State management:** Cần architecture rõ ràng cho multiplayer
- **Error handling:** Quan trọng cho user experience
- **Performance optimization:** Cần optimize từ đầu

#### 11.2.2 Development Process
- **Component architecture:** Modular design giúp maintain dễ dàng
- **TypeScript benefits:** Type safety giảm bugs đáng kể
- **Testing importance:** Unit tests giúp refactor an toàn
- **Documentation value:** Code comments và docs quan trọng

### 11.3 Project Impact

#### 11.3.1 Technical Skills Developed
- **Full-stack development:** Frontend + Backend integration
- **Real-time applications:** WebSocket programming
- **Game development:** Game logic và state management
- **UI/UX design:** Modern web interface design

#### 11.3.2 Business Value
- **Scalable architecture:** Có thể mở rộng thành sản phẩm thương mại
- **Modern tech stack:** Sử dụng công nghệ hiện đại
- **Production ready:** Code quality đạt chuẩn production
- **User-focused:** Thiết kế tập trung vào trải nghiệm người dùng

---

## 12. TÀI LIỆU THAM KHẢO

### 12.1 Technical Documentation
- [React Documentation](https://react.dev/)
- [Socket.IO Documentation](https://socket.io/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

### 12.2 Game Rules Reference
- [Official UNO Rules](https://www.unorules.com/)
- [UNO Card Game Variations](https://en.wikipedia.org/wiki/Uno_(card_game))

### 12.3 Development Tools
- [Vite Documentation](https://vitejs.dev/)
- [ESLint Configuration](https://eslint.org/docs/)
- [Node.js Best Practices](https://nodejs.org/en/docs/)

---

**Ngày tạo báo cáo:** {new Date().toLocaleDateString('vi-VN')}  
**Phiên bản:** 1.0  
**Tác giả:** Development Team  
**Trạng thái:** Hoàn thành  

---

*Báo cáo này cung cấp cái nhìn tổng quan chi tiết về dự án UNO Online Game, bao gồm thiết kế, implementation, và hướng phát triển tương lai.*