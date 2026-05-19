const fs = require('fs');
const path = require('path');

const GITHUB_USERNAME = 'Adibamrullah1';
const FULL_NAME = 'Mohammad Adib Amrullah';
const ROOT_DIR = 'c:\\laragon\\www';

// Direktori yang diabaikan agar tidak terlalu lama / salah deteksi
const IGNORED_DIRS = [
    'node_modules', 'vendor', '.git', '.github', '.next', '.vscode', '.idea',
    'platforms', 'platform-tools', 'ndk', 'licenses', 'cmake', 'cmdline-tools', 
    'build-tools', 'flutter', 'ReadmeProfle', 'Profile', '.temp'
];

// Map tech stack ke badge URL dari shields.io
const BADGES = {
    'Next.js': 'https://img.shields.io/badge/Next-black?style=for-the-badge&logo=next.js&logoColor=white',
    'React': 'https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB',
    'Laravel': 'https://img.shields.io/badge/laravel-%23FF2D20.svg?style=for-the-badge&logo=laravel&logoColor=white',
    'Flutter': 'https://img.shields.io/badge/Flutter-%2302569B.svg?style=for-the-badge&logo=Flutter&logoColor=white',
    'Tailwind CSS': 'https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white',
    'Prisma': 'https://img.shields.io/badge/Prisma-3982CE?style=for-the-badge&logo=Prisma&logoColor=white',
    'TypeScript': 'https://img.shields.io/badge/typescript-%23007ACC.svg?style=for-the-badge&logo=typescript&logoColor=white',
    'JavaScript': 'https://img.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E',
    'PHP': 'https://img.shields.io/badge/php-%23777BB4.svg?style=for-the-badge&logo=php&logoColor=white',
    'MySQL': 'https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white',
    'PostgreSQL': 'https://img.shields.io/badge/postgresql-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white',
};

// Fungsi rekursif untuk mencari proyek berdasarkan file konfigurasi
function findProjects(dir, depth = 0, maxDepth = 2) {
    if (depth > maxDepth) return [];
    
    let projects = [];
    let items = [];
    try {
        items = fs.readdirSync(dir, { withFileTypes: true });
    } catch (e) {
        return [];
    }

    const hasPackageJson = items.some(i => i.name === 'package.json');
    const hasComposerJson = items.some(i => i.name === 'composer.json');
    const hasPubspecYaml = items.some(i => i.name === 'pubspec.yaml');

    if (hasPackageJson || hasComposerJson || hasPubspecYaml) {
        // Ini adalah folder proyek!
        projects.push(analyzeProject(dir, hasPackageJson, hasComposerJson, hasPubspecYaml));
        return projects; // Jangan cari lebih dalam di dalam proyek ini
    }

    // Cari ke dalam sub-folder
    for (const item of items) {
        if (item.isDirectory() && !IGNORED_DIRS.some(ignored => item.name.includes(ignored) || item.name.startsWith('.'))) {
            projects = projects.concat(findProjects(path.join(dir, item.name), depth + 1, maxDepth));
        }
    }

    return projects;
}

// Menganalisis file proyek untuk menentukan Tech Stack
function analyzeProject(dir, hasPackageJson, hasComposerJson, hasPubspecYaml) {
    const projectName = path.basename(dir);
    const stack = new Set();
    
    if (hasPackageJson) {
        try {
            const pkg = JSON.parse(fs.readFileSync(path.join(dir, 'package.json'), 'utf-8'));
            const deps = { ...(pkg.dependencies || {}), ...(pkg.devDependencies || {}) };
            
            if (deps['next']) stack.add('Next.js');
            if (deps['react']) stack.add('React');
            if (deps['tailwindcss']) stack.add('Tailwind CSS');
            if (deps['prisma']) stack.add('Prisma');
            if (deps['typescript']) stack.add('TypeScript');
            else stack.add('JavaScript');
            
        } catch(e) {}
    }
    
    if (hasComposerJson) {
        try {
            const comp = JSON.parse(fs.readFileSync(path.join(dir, 'composer.json'), 'utf-8'));
            const deps = { ...(comp.require || {}), ...(comp['require-dev'] || {}) };
            
            if (deps['laravel/framework']) stack.add('Laravel');
            stack.add('PHP');
            
        } catch(e) {}
    }
    
    if (hasPubspecYaml) {
        stack.add('Flutter');
    }

    // Cek file spesifik
    if (fs.existsSync(path.join(dir, 'tailwind.config.js')) || fs.existsSync(path.join(dir, 'tailwind.config.ts'))) {
        stack.add('Tailwind CSS');
    }
    
    if (fs.existsSync(path.join(dir, '.env'))) {
        const envContent = fs.readFileSync(path.join(dir, '.env'), 'utf-8');
        if (envContent.includes('mysql')) stack.add('MySQL');
        if (envContent.includes('pgsql') || envContent.includes('postgres')) stack.add('PostgreSQL');
    }

    return {
        name: projectName,
        path: dir,
        stack: Array.from(stack)
    };
}

// Format judul agar lebih rapi (misal: "e-procurement" -> "E-Procurement")
function formatTitle(str) {
    return str.split(/[-_]/).map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' ');
}

// Mulai membuat README.md
function generateReadme() {
    console.log("🔍 Memindai folder proyek lokal...");
    const projects = findProjects(ROOT_DIR);
    
    console.log(`✅ Ditemukan ${projects.length} proyek!`);
    
    // Urutkan proyek (yang punya stack lebih banyak di atas)
    projects.sort((a, b) => b.stack.length - a.stack.length);

    let portfolioMarkdown = '## 💻 Local Projects Portfolio\n\nIni adalah proyek-proyek yang terdeteksi secara otomatis di workspace lokal saya.\n\n<div align="center">\n\n';
    
    projects.slice(0, 10).forEach(project => { // Tampilkan max 10 proyek terbaik
        const badges = project.stack.map(s => {
            return BADGES[s] ? '<img src="' + BADGES[s] + '" alt="' + s + '">' : '*' + s + '*';
        }).join(' ');
        
        portfolioMarkdown += '### 🚀 ' + formatTitle(project.name) + '\n';
        portfolioMarkdown += '> ' + (badges || 'Tech Stack: (Automated Detection)') + '\n\n';
    });
    
    portfolioMarkdown += '</div>\n';

    const readmeContent = `
<h1 align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&size=30&pause=1000&color=2563EB&center=true&vCenter=true&width=800&lines=Halo%2C+Saya+${FULL_NAME.replace(/ /g, '+')}!+👋;Software+Engineer+%7C+Tech+Enthusiast;Selamat+Datang+di+Profil+GitHub+Saya!" alt="Typing SVG" />
</h1>

<p align="center">
  <a href="https://github.com/${GITHUB_USERNAME}">
    <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=200&section=header&text=${encodeURIComponent(FULL_NAME)}&fontSize=50&fontAlignY=35&animation=fadeIn&fontColor=ffffff" alt="Header" />
  </a>
</p>

## 🚀 Tentang Saya
- 👨‍💻 Saya adalah seorang Software Developer yang antusias dalam membangun aplikasi web dan mobile.
- 💡 Saya tertarik dengan pengembangan **Frontend**, **Backend**, dan **Mobile Apps**.
- 🛠️ Tech Stack utama saya meliputi **Laravel, Next.js, React, dan Flutter**.
- 🤖 Selalu bersemangat untuk belajar teknologi baru dan memecahkan masalah dengan kode!

<br>

## 📊 GitHub Analytics & Activity
> Data ini diperbarui secara **otomatis** oleh GitHub.

<div align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=${GITHUB_USERNAME}&show_icons=true&theme=tokyonight&hide_border=true&count_private=true" width="48%" />
  <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=${GITHUB_USERNAME}&layout=compact&theme=tokyonight&hide_border=true" width="48%" />
</div>

<br>

<div align="center">
  <img src="https://github-readme-streak-stats.herokuapp.com/?user=${GITHUB_USERNAME}&theme=tokyonight&hide_border=true" width="100%" />
</div>

<br>

### 📈 Activity Graph
<div align="center">
  <img src="https://github-readme-activity-graph.vercel.app/graph?username=${GITHUB_USERNAME}&theme=tokyo-night&hide_border=true&area=true" width="100%" />
</div>

<br>

${portfolioMarkdown}

<br>

## 🏆 GitHub Trophies
<div align="center">
  <a href="https://github.com/ryo-ma/github-profile-trophy">
    <img src="https://github-profile-trophy.vercel.app/?username=${GITHUB_USERNAME}&theme=tokyonight&column=6&no-frame=true&margin-w=15" alt="Trophy" />
  </a>
</div>

<br>

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=100&section=footer" alt="Footer" />
</p>
`;

    fs.writeFileSync(path.join(__dirname, 'README.md'), readmeContent);
    console.log("✅ Berhasil membuat README.md yang mengagumkan!");
}

generateReadme();
