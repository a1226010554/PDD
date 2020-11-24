# PDD
作者：lalala
链接：https://www.zhihu.com/question/19841574/answer/665411848
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

电商系统设计之——商品SPU、SPU前言    在一个标准的电商系统中，SKU的设计肯定是必不可少的。这里定义的“标准”二字，是指在这个系统中，可以实现对现实生活中大部分商品的维护以及售卖。可是我们知道，现实生活中的商品是形形色色的，几乎任何一个商品，都会有不同的规格参数，这些规格以不同的方式组合，对应着不同的价格和库存；我们的系统不可能会针对每一种组合去建一个商品，我相信大部分电商系统也不会这样设计，那样商品维护起来成本就太高了。那么，我们在商品的模型上，该如何设计呢？面对这样的问题，就诞生了SPU和SKU的概念。SPU与SKUSPU概念SPU(Standard Product Unit)：标准化产品单元。是商品信息聚合的最小单位，是一组可复用、易检索的标准化信息的集合，该集合描述了一个产品的特性。通俗点讲，属性值、特性相同的商品就可以称为一个SPU。SKU概念SKU（Stock Keeping Unit）：库存量单位。即库存进出计量的基本单元，可以是以件，盒，托盘等为单位。SPU与SKU的关系在商品模型设计中，要表述一个完整的商品，应该包含SPU属性和SKU属性。SPU属性与商品的库存无关，仅仅用来描述一个商品，也可以称为“描述属性”。SKU属性与商品的库存和价格有关，每一种SKU都有自己的库存和价格，也可以称为“销售属性”。从概念上理解SPU和SKU之间的关系还比较抽象，下面我们从一个例子入手，他们理解起来就比较简单了。    我们拿iPhone X手机举例子，它至少拥有“颜色”、“内存”，这两个属性；然后"颜色"属性有"深灰色"和“银色”可选项；“内存”属性有"64G"和“256G”可选项，那么这个iPhone X通过这些属性的组合，得到4个规格不同的iPhone X，分别是：深灰色64G的iPhone X银色64G的iPhone X深灰色256G的iPhone X银色256G的iPhone X    在这个商品里我们就可以把iPhone X抽象成1个SPU，4个组合分别抽象成4个SKU。上图中并没有直观显示出SPU属性，但是它肯定是存在的。就好比，这个iPhone X的“产地”，“重量”，“电池容量”这些都属于SPU属性，这些属性信息，可能会在商品详情里面看到。数据库设计    上面iPhone X只是我们举已一个简单的例子，一个商品的SPU属性和SKU属性，应该是允许商家自己定义的，并且可以无限扩充的。    这些属性并不特指商品自身所带的属性，商家还可以根据实际情况和营销策略去赋予商品一些属性。比如我们在淘宝或者京东肯定见过“套餐一”、“套餐二”类似这样的属性。为了保持商品的通用性和扩展性，我们的库表必须设计成可动态扩展的结构，这里的核心思想就是将属性名称和属性值拆分在不同的表中维护。关于SPU与SKU的设计有很多种方案，以下是其中一种实现方案。数据模型这种设计是将SPU属性和SKU属性都挂在商品的类目下，这样做的好处是方便商家维护商品属性，不用针对每一个商品，都建一套属于自己的SPU和SKU，因为同一类目下的商品，SPU属性项、SKU属性项几乎是一样的，不一样的只是属性值。上面数数据库模型中sku_attribute表和spu_attribute表都只是维护属性的名称、排序和展示样式而已，最终决定一个商品属性的数据是在goods_spu_attribute_value表和goods_sku_option_group表。附上相关建表SQLgoods_category 商品类目表spu_attribute SPU属性表sku_attribute SKU属性表sku_option SKU可选项表goods 商品主表goods_spu_attribute_value 商品SPU属性表goods_sku 商品SKU表goods_sku_option_group 商品SKU选项组合表SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for goods
-- ----------------------------
DROP TABLE IF EXISTS `goods`;
CREATE TABLE `goods`  (
  `id` varchar(32) NOT NULL COMMENT '商品ID',
  `category_id` varchar(32) DEFAULT NULL COMMENT '类目叶子节点ID',
  `merchant_id` varchar(32) DEFAULT NULL COMMENT '商户ID',
  `brand_id` varchar(32) DEFAULT NULL COMMENT '品牌ID',
  `goods_code` varchar(32) DEFAULT NULL COMMENT '商品编号',
  `front_name` varchar(255) DEFAULT NULL COMMENT '前端名称',
  `back_name` varchar(255) DEFAULT NULL COMMENT '后端名称',
  `goods_type` tinyint(1) DEFAULT NULL COMMENT '商品类型（0单品，1套餐）',
  `goods_status` tinyint(1) DEFAULT NULL COMMENT '状态（-1下架，0新建，1上架）',
  `goods_desc` varchar(512) DEFAULT NULL COMMENT '描述',
  `cover_pic` varchar(255) DEFAULT NULL COMMENT '封面图',
  `key_word` varchar(32) DEFAULT NULL COMMENT '关键字',
  `create_user` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_date` datetime(0) DEFAULT NULL COMMENT '创建时间',
  `update_user` varchar(32) DEFAULT NULL COMMENT '修改人',
  `update_date` datetime(0) DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '商品主表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for goods_category
-- ----------------------------
DROP TABLE IF EXISTS `goods_category`;
CREATE TABLE `goods_category`  (
  `id` varchar(32) NOT NULL COMMENT '类目ID',
  `name` varchar(32) DEFAULT NULL COMMENT '类目名称',
  `level` int(11) DEFAULT NULL COMMENT '类目级别（1一级，2二级，3三级，...）',
  `has_child` tinyint(1) DEFAULT NULL COMMENT '是否存在子节点（0否，1是）',
  `parent_id` varchar(32) DEFAULT NULL COMMENT '父节点ID',
  `create_user` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_date` datetime(0) DEFAULT NULL COMMENT '创建时间',
  `update_user` varchar(32) DEFAULT NULL COMMENT '修改人',
  `update_date` datetime(0) DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '商品类目表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for goods_sku
-- ----------------------------
DROP TABLE IF EXISTS `goods_sku`;
CREATE TABLE `goods_sku`  (
  `id` varchar(32) NOT NULL COMMENT '商品SKUID',
  `goods_id` varchar(32) DEFAULT NULL COMMENT '商品ID',
  `sku_code` varchar(32) DEFAULT NULL COMMENT 'SKU码',
  `cost_price` int(11) DEFAULT NULL COMMENT '入库价（分）',
  `sale_price` int(11) DEFAULT NULL COMMENT '销售价（分）',
  `disc_price` int(11) DEFAULT NULL COMMENT '折扣价（分）',
  `stock` int(11) DEFAULT NULL COMMENT '库存',
  `create_user` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_date` datetime(0) DEFAULT NULL COMMENT '创建时间',
  `update_user` varchar(32) DEFAULT NULL COMMENT '修改人',
  `update_date` datetime(0) DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '商品SKU表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for goods_sku_option_group
-- ----------------------------
DROP TABLE IF EXISTS `goods_sku_option_group`;
CREATE TABLE `goods_sku_option_group`  (
  `id` varchar(32) NOT NULL COMMENT 'ID',
  `goods_id` varchar(32) DEFAULT NULL COMMENT '商品ID',
  `sku_id` varchar(32) DEFAULT NULL COMMENT 'SKUID',
  `sku_option_id` varchar(32) DEFAULT NULL COMMENT 'SKU选项ID',
  `create_user` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_date` datetime(0) DEFAULT NULL COMMENT '创建时间',
  `update_user` varchar(32) DEFAULT NULL COMMENT '修改人',
  `update_date` datetime(0) DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '商品SKU选项组合表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for goods_spu_attribute_value
-- ----------------------------
DROP TABLE IF EXISTS `goods_spu_attribute_value`;
CREATE TABLE `goods_spu_attribute_value`  (
  `id` varchar(32) NOT NULL COMMENT 'SPU属性值ID',
  `goods_id` varchar(32) DEFAULT NULL COMMENT '商品ID',
  `spu_attribute_id` varchar(32) DEFAULT NULL COMMENT '私有属性ID',
  `spu_attribute_value` varchar(512) DEFAULT NULL COMMENT '属性值',
  `value_type` tinyint(1) DEFAULT NULL COMMENT '属性值类型（0普通字符串，1时间戳格式，2时间格式字符串(yyyy-MM-dd HH:mm:ss)，...）',
  `value_desc` varchar(255) DEFAULT NULL COMMENT '属性值说明',
  `sort` int(11) DEFAULT NULL COMMENT '排序（1最前）',
  `create_user` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_date` datetime(0) DEFAULT NULL COMMENT '创建时间',
  `update_user` varchar(32) DEFAULT NULL COMMENT '修改人',
  `update_date` datetime(0) DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '商品SPU属性值表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sku_attribute
-- ----------------------------
DROP TABLE IF EXISTS `sku_attribute`;
CREATE TABLE `sku_attribute`  (
  `id` varchar(32) NOT NULL COMMENT 'SKU属性ID',
  `category_id` varchar(32) DEFAULT NULL COMMENT '类目叶子节点ID',
  `front_name` varchar(32) DEFAULT NULL COMMENT '属性名称（前端）',
  `back_name` varchar(32) DEFAULT NULL COMMENT '属性名称（后端）',
  `attr_desc` varchar(512) DEFAULT NULL COMMENT '属性说明',
  `sort` int(11) DEFAULT NULL COMMENT '排序（1最前）',
  `create_user` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_date` datetime(0) DEFAULT NULL COMMENT '创建时间',
  `update_user` varchar(32) DEFAULT NULL COMMENT '修改人',
  `update_date` datetime(0) DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = 'SKU属性表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sku_option
-- ----------------------------
DROP TABLE IF EXISTS `sku_option`;
CREATE TABLE `sku_option`  (
  `id` varchar(32) NOT NULL COMMENT 'SKU可选项ID',
  `sku_attribute_id` varchar(32) DEFAULT NULL COMMENT 'SKU属性ID',
  `option_name` varchar(32) DEFAULT NULL COMMENT '可选项名称',
  `option_desc` varchar(512) DEFAULT NULL COMMENT '可选项说明',
  `sort` int(11) DEFAULT NULL COMMENT '排序（1最前）',
  `create_user` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_date` datetime(0) DEFAULT NULL COMMENT '创建时间',
  `update_user` varchar(32) DEFAULT NULL COMMENT '修改人',
  `update_date` datetime(0) DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = 'SKU可选项表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for spu_attribute
-- ----------------------------
DROP TABLE IF EXISTS `spu_attribute`;
CREATE TABLE `spu_attribute`  (
  `id` varchar(32) NOT NULL COMMENT 'SPU属性ID',
  `category_id` varchar(32) DEFAULT NULL COMMENT '类目叶子节点ID',
  `front_name` varchar(32) DEFAULT NULL COMMENT '属性名称（前端）',
  `back_name` varchar(32) DEFAULT NULL COMMENT '属性名称（后端）',
  `input_style` int(11) DEFAULT NULL COMMENT '属性值输入样式（0文本输入框，1时间控件，2下拉选项，...）',
  `attr_desc` varchar(512) DEFAULT NULL COMMENT '属性说明',
  `sort` int(11) DEFAULT NULL COMMENT '排序（1最前）',
  `create_user` varchar(32) DEFAULT NULL COMMENT '创建人',
  `create_date` datetime(0) DEFAULT NULL COMMENT '创建时间',
  `update_user` varchar(32) DEFAULT NULL COMMENT '修改人',
  `update_date` datetime(0) DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = 'SPU属性表' ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;
