# 半边数据结构的设计与实现

## 引言

半边数据结构是一种用于表示和处理多边形网格的数据结构。在半边数据结构中，每个面都由一个半边开始，通过遍历相邻的半边可以得到该面的所有顶点和边。每个半边都包含了指向下一个半边、相反方向的半边、所属的面和起始顶点的指针。这种结构的设计使得在进行各种几何操作时更加高效，如查找相邻面、遍历边、寻找邻近顶点等。

半边数据结构的主要优点是在进行拓扑修改时效率高，例如添加、删除或分割面、边或顶点。它还可以有效地存储和查询与顶点、边或面相关联的属性信息，如法向量、纹理坐标等。

## 数据结构设计

```c++
class MHalfedge {
private:
	int index_;

	MVertex* v_;
	MEdge* e_;
	MFace* f_;

	MHalfedge* next_, * prev_;	// 后继，前驱
	MHalfedge* twin_;	// 对边

public:
	MHalfedge() : index_(-1), next_(nullptr), prev_(nullptr), twin_(nullptr),
		v_(nullptr), e_(nullptr), f_(nullptr) {}
	MHalfedge(MHalfedge* next, MHalfedge* prev, MHalfedge* twin, MVertex* v, MEdge* e, MFace* f)
		: index_(-1), next_(next), prev_(prev), twin_(twin), v_(v), e_(e), f_(f) {}

	~MHalfedge() { index_ = -1; }

public:
	MHalfedge* const next() { return next_; }
	MHalfedge* const prev() { return prev_; }
	MHalfedge* const twin() { return twin_; }
	MVertex* const fromVertex() { return v_; }
	MVertex* const toVertex() { return next()->fromVertex(); }
	MEdge* const edge() { return e_; }
	MFace* const face() { return f_; }

	MHalfedge* const rotateNext() { return twin()->next(); }
	MHalfedge* const rotatePrev() { return prev()->twin(); }

	const MHalfedge* const next() const { return next_; }
	const MHalfedge* const prev() const { return prev_; }
	const MHalfedge* const twin() const { return twin_; }
	const MVertex* const fromVertex() const { return v_; }
	const MVertex* const toVertex() const { return next()->fromVertex(); }
	const MEdge* const edge() const { return e_; }
	const MFace* const face() const { return f_; }

	void setNext(MHalfedge* next) { next_ = next; }
	void setPrev(MHalfedge* prev) { prev_ = prev; }
	void setTwin(MHalfedge* twin) { twin_ = twin; }
	void setVertex(MVertex* vertex) { v_ = vertex; }
	void setEdge(MEdge* edge) { e_ = edge; }
	void setFace(MFace* face) { f_ = face; }

	bool isBoundary() const { return f_ == nullptr; }

	int index() { return index_; }

	void set_index(int index) { index_ = index; }
};

```



## 读取obj文件

```c++
bool OBJReader::loadMeshFromOBJ(std::istream& _in, PolyMesh* mesh) {
	string line;
	stringstream ss, data, tmp;
	double x, y, z;
	string keyword;

	while (_in && !_in.eof()) {
		std::getline(_in, line);
		if (line.size() == 0 || line[0] == '#' || isspace(line[0])) continue;

		ss.str(line);
		ss.clear();
		ss >> keyword;
		if (keyword == "v") {
			ss >> x >> y >> z;
			mesh->addVertex(x, y, z);
		}
		else if (keyword == "f") {
			string faceline;
			std::getline(ss, faceline);

			data.str(faceline);
			data.clear();

			string vertex;
			std::vector<size_t> faceVertices;
			for (int i = 0; i < 3; i++) {
				data >> vertex;

				int v;
				auto found = vertex.find("/");
				if (found != string::npos) {
					tmp.str(vertex.substr(0, found));
				}
				else {
					tmp.str(vertex);
				}
				tmp.clear();
				tmp >> v;

				faceVertices.push_back(v - 1);
			}
			mesh->addFace(faceVertices);
		}
	}

	return true;
}
```



## 调整结构关系

```c++
MFace * PolyMesh::addFace(std::vector<MVertex*>&v) {
    size_t v_size = v.size();

    MFace* face = newFace();

    std::vector<bool> is_edge_new(v_size, false);
    std::vector<bool> is_needc(v_size, false);

    MHalfedge* he1 = nullptr;
    MHalfedge* he2 = nullptr;
    std::vector<MHalfedge*> edges;
    for (size_t i = 0; i < v_size; i++) {
        size_t j = (i + 1) % v_size;

        MEdge* e = edgeBetween(v[i], v[j]);
        if (e == nullptr) {
            e = addEdge(v[i], v[j]);
            is_edge_new[i] = true;
        }

        if (e->halfEdge()->fromVertex() == v[i]) {
            he1 = e->halfEdge();
            he2 = e->halfEdge()->twin();
        }
        else {
            he1 = e->halfEdge()->twin();
            he2 = e->halfEdge();
        }

        if (face->halfEdge() == nullptr) {
            face->setHalfedge(he1);
        }
        he1->setFace(face);
        edges.push_back(he1);
    }

    MHalfedge* he_ip, * he_in;
    MHalfedge* he_bp, * he_bn;
    for (size_t i = 0; i < v_size; i++) {
        size_t j = (i + 1) % v_size;

        MVertex* vh = v[j];
        he_ip = edges[i];
        he_in = edges[j];

        assert(he_ip != nullptr);
        assert(he_in != nullptr);

        size_t id = 0;
        if (is_edge_new[i]) id |= 1;
        if (is_edge_new[j]) id |= 2;

        MHalfedge* he_op = he_in->twin();
        MHalfedge* he_on = he_ip->twin();

        switch (id) {
        case 1: // prev is new, next is old
            he_bp = he_in->prev();
            he_bp->setNext(he_on);
            he_on->setPrev(he_bp);
            vh->setHalfedge(he_on);
            break;

        case 2: // next is new, prev is old
            he_bn = he_ip->next();
            he_op->setNext(he_bn);
            he_bn->setPrev(he_op);
            vh->setHalfedge(he_bn);
            break;

        case 3: // both are new
            if (vh->halfEdge() == nullptr)
            {
                vh->setHalfedge(he_on);
                he_op->setNext(he_on);
                he_on->setPrev(he_op);
            }
            else
            {
                he_bn = vh->halfEdge();
                he_bp = he_bn->prev();

                he_bp->setNext(he_on);
                he_on->setPrev(he_bp);
                he_op->setNext(he_bn);
                he_bn->setPrev(he_op);
            }
            break;
        case 0:
            is_needc[j] = true;
        }

        he_ip->setNext(he_in);
        he_in->setPrev(he_ip);
    }

    for (size_t i = 0; i < v_size; i++) {
        if (is_needc[i]) { // 都是旧边调整出去的第一条边，尽量为边界边
            v[i]->adjustOutgoingHalfedge();
        }
    }

    return face;
}
```



## 结语

[在此](https://github.com/s1san/sbr/tree/main/HalfedgeImpl)查看项目代码

## References

[三角网格半边数据结构设计与实现](https://blog.csdn.net/chenbb1989/article/details/123774576)

[geometry-central](https://github.com/nzfeng/geometry-central/tree/89121a68556fbb8479aad811c716b9839f318b52)