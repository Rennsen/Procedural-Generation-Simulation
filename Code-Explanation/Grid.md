### `Grid.cs`
```csharp
public class Grid : MonoBehaviour
{
    public float scale = .1f;
    public int size = 150;

    Cell[,] grid;
```
- These lines define the `Grid` class, inheriting from `MonoBehaviour`, which means it can be attached to a Unity game object.
- `scale` determines the frequency of Perlin noise. Smaller values create more detailed noise.
- `size` specifies the dimensions of the grid.
- `grid` is a 2D array of `Cell` objects representing the grid cells.

```csharp
void Start()
{
    float[,] noiseMap = new float[size, size];
    (float xOffset, float yOffset) = (Random.Range(-10000f, 10000f), Random.Range(-10000f, 10000f));
```
- `Start()` is a Unity callback method called when the script is initialized.
- `noiseMap` is a 2D array to store Perlin noise values for each grid cell.
- `(float xOffset, float yOffset)` declares two float variables using C# tuple syntax to store random offsets for Perlin noise generation.

```csharp
for (int y = 0; y < size; y++)
{
    for (int x = 0; x < size; x++)
    {
        float noiseValue = Mathf.PerlinNoise(x * scale + xOffset, y * scale + yOffset);
        noiseMap[x, y] = noiseValue;
    }
}
```
- Nested loops iterate over each grid cell to calculate Perlin noise values.
- `Mathf.PerlinNoise` generates a Perlin noise value based on the grid cell coordinates and offsets.
- The noise value is stored in `noiseMap` at the corresponding grid cell position.

```csharp
float[,] falloffMap = new float[size, size];
for (int y = 0; y < size; y++)
{
    for (int x = 0; x < size; x++)
    {
        float xv = x / (float)size * 2 - 1;
        float yv = y / (float)size * 2 - 1;
        float v = Mathf.Max(Mathf.Abs(xv), Mathf.Abs(yv));
        falloffMap[x, y] = Mathf.Pow(v, 3f) / (Mathf.Pow(v, 3f) + Mathf.Pow(2.2f - 2.2f * v, 3f));
    }
}
```
- Another 2D array, `falloffMap`, is created to simulate smooth transitions between different terrain types.
- Nested loops calculate falloff values for each grid cell based on their distance from the center.
- `Mathf.Pow` calculates the power of a number to create smooth falloff.
```csharp
grid = new Cell[size, size];
for (int y = 0; y < size; y++)
{
    for (int x = 0; x < size; x++)
    {
        float noiseValue = noiseMap[x, y];
        noiseValue -= falloffMap[x, y];
        bool isWater = noiseValue < waterLevel;
        Cell cell = new Cell(isWater);
        grid[x, y] = cell;
    }
}
```
- The `grid` array is initialized with `Cell` objects based on noise and falloff maps.
- Nested loops iterate over each grid cell.
- `noiseValue` is retrieved from the `noiseMap` array.
- `falloffMap` values are subtracted from `noiseValue` to create smoother transitions.
- A boolean `isWater` is determined based on whether `noiseValue` is less than `waterLevel`.
- A new `Cell` object is created with the determined `isWater` value and added to the grid.

```csharp
void DrawTerrainMesh(Cell[,] grid)
{
    Mesh mesh = new Mesh();
    List<Vector3> vertices = new List<Vector3>();
    List<int> triangles = new List<int>();
    List<Vector2> uvs = new List<Vector2>();
```
- `DrawTerrainMesh` method initializes lists to store vertices, triangles, and UV coordinates for the terrain mesh.

```csharp
for (int y = 0; y < size; y++)
{
    for (int x = 0; x < size; x++)
    {
        Cell cell = grid[x, y];
        if (!cell.isWater)
        {
            Vector3 a = new Vector3(x - .5f, 0, y + .5f);
            Vector3 b = new Vector3(x + .5f, 0, y + .5f);
            Vector3 c = new Vector3(x - .5f, 0, y - .5f);
            Vector3 d = new Vector3(x + .5f, 0, y - .5f);
            Vector2 uvA = new Vector2(x / (float)size, y / (float)size);
            Vector2 uvB = new Vector2((x + 1) / (float)size, y / (float)size);
            Vector2 uvC = new Vector2(x / (float)size, (y + 1) / (float)size);
            Vector2 uvD = new Vector2((x + 1) / (float)size, (y + 1) / (float)size);
            Vector3[] v = new Vector3[] { a, b, c, b, d, c };
            Vector2[] uv = new Vector2[] { uvA, uvB, uvC, uvB, uvD, uvC };
```
- Nested loops iterate over each grid cell.
- Vertices (`a`, `b`, `c`, `d`) and UV coordinates (`uvA`, `uvB`, `uvC`, `uvD`) are calculated for each quad representing a grid cell.
- `Vector3` represents 3D points in space, while `Vector2` represents 2D points (UV coordinates).
- `x` and `y` are used to determine the position of vertices and UV coordinates.

```csharp
for (int k = 0; k < 6; k++)
{
    vertices.Add(v[k]);
    triangles.Add(triangles.Count);
    uvs.Add(uv[k]);
}
```
- A loop iterates over the vertices of each quad.
- Vertices, triangles, and UV coordinates are added to their respective lists.
- `triangles.Add(triangles.Count)` adds the current index to the triangles list, forming triangles from the vertices.

```csharp
mesh.vertices = vertices.ToArray();
mesh.triangles = triangles.ToArray();
mesh.uv = uvs.ToArray();
mesh.RecalculateNormals();
```
- Lists of vertices, triangles, and UV coordinates are converted to arrays and assigned to the mesh.
- `RecalculateNormals()` recalculates the normals of the mesh to ensure proper lighting in Unity.

```csharp
MeshFilter meshFilter = gameObject.AddComponent<MeshFilter>();
meshFilter.mesh = mesh;

MeshRenderer meshRenderer = gameObject.AddComponent<MeshRenderer>();
```
- MeshFilter component is added to the game object to render the mesh.
- MeshRenderer component is added to render the mesh with materials.

```csharp
void DrawEdgeMesh(Cell[,] grid)
{
    Mesh mesh = new Mesh();
    List<Vector3> vertices = new List<Vector3>();
    List<int> triangles = new List<int>();
```
- Similar to `DrawTerrainMesh`, this method initializes lists for vertices and triangles.

```csharp
if (!cell.isWater)
{
    if (x > 0)
    {
        Cell left = grid[x - 1, y];
        if (left.isWater)
        {
            Vector3 a = new Vector3(x - .5f, 0, y + .5f);
            Vector3 b = new Vector3(x - .5f, 0, y - .5f);
            Vector3 c = new Vector3(x - .5f, -1, y + .5f);
            Vector3 d = new Vector3(x - .5f, -1, y - .5f);
            Vector3[] v = new Vector3[] { a, b, c, b, d, c };
```
- This section of code checks if the current cell is not water and if the neighboring cell to the left is water.
- If conditions are met, vertices for the left edge of the current cell are created, representing the transition from land to water.

```csharp
Vector3[] v = new Vector3[] { a, b, c, b, d, c };
for (int k = 0; k < 6; k++)
{
    vertices.Add(v[k]);
    triangles.Add(triangles.Count);
}
```
- Similar to the terrain mesh, vertices and triangles are added to their respective lists for the edge mesh.

```csharp
GameObject edgeObj = new GameObject("Edge");
edgeObj.transform.SetParent(transform);

MeshFilter meshFilter = edgeObj.AddComponent<MeshFilter>();
meshFilter.mesh = mesh;

MeshRenderer meshRenderer = edgeObj.AddComponent<MeshRenderer>();
meshRenderer.material = edgeMaterial;
```
- A new game object named "Edge" is created to hold the edge mesh.
- MeshFilter and MeshRenderer components are added to render the mesh with the specified material.

```csharp
void DrawTexture(Cell[,] grid)
{
    Texture2D texture = new Texture2D(size, size);
    Color[] colorMap = new Color[size * size];
```
- This method creates a texture to represent the grid's terrain.
- A 2D texture and an array to store color data are initialized.

```csharp
for (int y = 0; y < size; y++)
{
    for (int x = 0; x < size; x++)
    {
        Cell cell = grid[x, y];
        if (cell.isWater)
            colorMap[y * size + x] = new Color(64 / 255f, 164 / 255f, 223 / 255f);
        else
            colorMap[y * size + x] = new Color(153 / 255f, 191 / 255f, 115 / 255f);
    }
}
```
- Nested loops iterate over each grid cell.
- Based on whether the cell is water or land, a color is assigned to the corresponding pixel in the color map.
- Water cells are assigned a blue color, while land cells are assigned a green color.

```csharp
texture.filterMode = FilterMode.Point;
texture.SetPixels(colorMap);
texture.Apply();
```
- `FilterMode.Point` ensures that pixels maintain their original sharpness when scaled.
- Color data is applied to the texture, and changes are applied immediately with `Apply()`.

```csharp
MeshRenderer meshRenderer = gameObject.GetComponent<MeshRenderer>();
meshRenderer.material = terrainMaterial;
meshRenderer.material.mainTexture = texture;
```
- MeshRenderer component of the main game object is accessed.
- The material for rendering terrain is assigned.
- The generated texture is set as the main texture of the material, giving the terrain its visual representation.

```csharp
void GenerateTrees(Cell[,] grid)
{
    float[,] noiseMap = new float[size, size];
    (float xOffset, float yOffset) = (Random.Range(-10000f, 10000f), Random.Range(-10000f, 10000f));
```
- This method generates trees on non-water cells of the grid.
- A noise map is initialized to determine tree density across the grid.
- Random offsets are generated to add variation to the noise map.

```csharp
for (int y = 0; y < size; y++)
{
    for (int x = 0; x < size; x++)
    {
        float noiseValue = Mathf.PerlinNoise(x * treeNoiseScale + xOffset, y * treeNoiseScale + yOffset);
        noiseMap[x, y] = noiseValue;
    }
}
```
- Nested loops iterate over each grid cell to calculate Perlin noise values for tree placement.
- `Mathf.PerlinNoise` generates noise values based on cell coordinates and offsets, determining tree density.

```csharp
for (int y = 0; y < size; y++)
{
    for (int x = 0; x < size; x++)
    {
        Cell cell = grid[x, y];
        if (!cell.isWater)
        {
            float v = Random.Range(0f, treeDensity);
            if (noiseMap[x, y] < v)
            {
                GameObject prefab = treePrefabs[Random.Range(0, treePrefabs.Length)];
                GameObject tree = Instantiate(prefab, transform);
                tree.transform.position = new Vector3(x, 0, y);
                tree.transform.rotation = Quaternion.Euler(0, Random.Range(0, 360f), 0);
                tree.transform.localScale = Vector3.one * Random.Range(.8f, 1.2f);
            }
        }
    }
}
```
- Nested loops iterate over each grid cell to place trees on non-water cells.
- Random values are generated to determine if a tree should be placed based on noise and tree density.
- If conditions are met, a random tree prefab is instantiated at the cell position with randomized rotation and scale.
