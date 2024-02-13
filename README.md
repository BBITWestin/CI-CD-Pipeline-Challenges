# My Docs

![Static Badge](https://img.shields.io/badge/------------%20HELLO%20WORLD%20-------------8A2BE2.svg?style=for-the-badge)

[![Commits][last-commits-shield]][last-commits-url]
[![Contributors][contributors-shield]][contributors-url]

[![Issues][issues-shield]][issues-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]

<html>
  <body>
    <script>
      (async () => {
        const response = await fetch('https://api.github.com/repos/BBITWestin/https://bbitwestin.github.io//contents/');
        const data = await response.json();
        let htmlString = '<ul>';
        
        for (let file of data) {
          htmlString += `<li><a href="${file.path}">${file.name}</a></li>`;
        }

        htmlString += '</ul>';
        document.getElementsByTagName('body')[0].innerHTML = htmlString;
      })()
    </script>

  <body>
</html>

## Contributing

1. Fork the Project
2. Create your Feature Branch
   ```sh
   git checkout -b feature/AmazingFeature
   ```
3. Commit & Push your Changes
   ```sh
   git add . ; git commit -m "Some AmazingFeature" ; git push
   ```
4. Open a Pull Request

## Links

[Emojis](https://gist.github.com/rxaviers/7360908)

[contributors-shield]: https://img.shields.io/github/contributors/BBITWestin/My-Docs.svg?style=for-the-badge
[contributors-url]: https://github.com/BBITWestin/My-Docs/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/BBITWestin/My-Docs.svg?style=for-the-badge
[forks-url]: https://github.com/BBITWestin/My-Docs/network/members
[last-commits-shield]: https://img.shields.io/github/last-commit/BBITWestin/My-Docs.svg?style=for-the-badge
[last-commits-url]: https://github.com/BBITWestin/My-Docs/commits/main/
[stars-shield]: https://img.shields.io/github/stars/BBITWestin/My-Docs.svg?style=for-the-badge
[stars-url]: https://github.com/BBITWestin/My-Docs/stargazers
[issues-shield]: https://img.shields.io/github/issues/BBITWestin/My-Docs.svg?style=for-the-badge
[issues-url]: https://github.com/BBITWestin/My-Docs/issues
