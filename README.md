<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Mini Reddit</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            background: #dae0e6;
            margin: 0;
            padding: 0;
        }

        header {
            background: #ff4500;
            color: white;
            padding: 15px;
            font-size: 24px;
            font-weight: bold;
        }

        .container {
            width: 600px;
            margin: 20px auto;
        }

        /* Formulaire */
        .post-form {
            background: white;
            padding: 15px;
            border-radius: 5px;
            margin-bottom: 20px;
        }

        .post-form input, .post-form textarea {
            width: 100%;
            padding: 8px;
            margin-bottom: 10px;
        }

        .post-form button {
            background: #ff4500;
            color: white;
            padding: 10px 15px;
            border: none;
            cursor: pointer;
            border-radius: 4px;
        }

        /* Post */
        .post {
            background: white;
            padding: 15px;
            border-radius: 5px;
            margin-bottom: 20px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .post img {
            max-width: 100%;
            margin-top: 10px;
            border-radius: 4px;
        }

        .title {
            font-weight: bold;
            font-size: 18px;
        }

        .content {
            margin-top: 5px;
        }
    </style>
</head>

<body>

<header>Mini Reddit</header>

<div class="container">

    <!-- Formulaire de poste -->
    <div class="post-form">
        <input type="text" id="title" placeholder="Titre du post">
        <textarea id="content" placeholder="Contenu du post"></textarea>
        <input type="text" id="imageUrl" placeholder="URL de l'image (optionnel)">
        <button onclick="addPost()">Publier</button>
    </div>

    <div id="posts"></div>

</div>

<script>
    function addPost() {
        const title = document.getElementById("title").value;
        const content = document.getElementById("content").value;
        const imageUrl = document.getElementById("imageUrl").value;

        if (!title || !content) {
            alert("Titre et contenu obligatoires !");
            return;
        }

        const postsDiv = document.getElementById("posts");

        // Création du post
        const post = document.createElement("div");
        post.className = "post";

        post.innerHTML = `
            <div class="title">${title}</div>
            <div class="content">${content}</div>
            ${imageUrl ? `<img src="${imageUrl}" alt="image du post">` : ""}
        `;

        postsDiv.prepend(post);

        // Effacer le formulaire
        document.getElementById("title").value = "";
        document.getElementById("content").value = "";
        document.getElementById("imageUrl").value = "";
    }
</script>

</body>
</html>
