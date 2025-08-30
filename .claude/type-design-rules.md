# Galaxy 项目类型设计规则

## 核心原则

### 1. 类型优先查找规则 🔍

**在创建任何新类型之前，必须按以下顺序检查：**

1. **检查 `src/types/` 目录**
   ```bash
   # 搜索相关类型
   find src/types -name "*.ts" -exec grep -l "接口名" {} \;
   ```

2. **检查现有组件和 Hook**
   ```bash
   # 搜索现有类型定义
   grep -r "interface.*接口名" src/
   ```

3. **检查第三方库类型**
   - 是否有现成的库类型可以扩展
   - 避免重复定义已有的标准类型

### 2. 类型命名和组织规范

#### 目录结构
```
src/types/
├── index.ts           # 统一导出
├── account/           # 账户相关
├── planet/            # 星球相关
├── game/              # 游戏逻辑
├── ui/                # UI 组件类型
└── api/               # API 响应类型
```

#### 命名规范
- **接口**: PascalCase，描述性名称
- **类型别名**: PascalCase + Type 后缀
- **枚举**: PascalCase + Enum 后缀

### 3. 类型兼容性策略

#### 标准类型 vs 组件特定类型

**标准类型** (`src/types/planet/index.ts`):
```typescript
interface Planet {
  id: string
  name: string
  createdBy: number
  x: number
  y: number
  size: number
  // ... 完整的业务字段
}
```

**组件适配类型** (`src/hooks/useUniverseCanvas.ts`):
```typescript
// 使用适配器模式
type CanvasPlanet = Pick<Planet, 'id' | 'name' | 'x' | 'y'> & {
  radius: number // 映射 size -> radius
  color: string // UI 特有属性
  rating?: number // 计算属性：基于 likes/dislikes
}
```

## 实施规则

### 强制检查流程

1. **新类型创建前**：
   - [ ] 搜索现有类型
   - [ ] 检查是否可复用
   - [ ] 评估是否需要适配器

2. **类型设计时**：
   - [ ] 优先扩展现有类型
   - [ ] 使用 Pick/Omit 创建子集
   - [ ] 明确标注适配关系

3. **Code Review 检查**：
   - [ ] 是否重复定义
   - [ ] 是否正确引用标准类型
   - [ ] 适配器是否必要

### 适配器模式应用

#### 何时使用适配器
- 标准类型字段名与组件需求不匹配
- 需要添加 UI 特有属性
- 需要计算属性或派生字段

#### 适配器实现模式
```typescript
// 1. 字段映射
type UIPlanet = Omit<Planet, 'size'> & {
  radius: number // size -> radius
}

// 2. 添加 UI 属性
type CanvasPlanet = Pick<Planet, 'id' | 'name' | 'x' | 'y'> & {
  radius: number
  color: string
  owner?: string
}

// 3. 计算属性
type PlanetWithRating = Planet & {
  rating: number // 基于 likes/dislikes 计算
}
```

## 重构指导

### 当前需要重构的文件
- `src/hooks/useUniverseCanvas.ts` - 星球类型适配

### 重构步骤
1. 导入标准 Planet 类型
2. 创建适配器类型
3. 更新数据转换逻辑
4. 添加类型文档

### 长期目标
- 所有组件优先使用标准类型
- 建立完整的类型适配器库
- 自动化类型一致性检查

---

**遵循这些规则将确保类型系统的一致性和可维护性。**
