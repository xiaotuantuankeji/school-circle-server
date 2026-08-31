# School Circle Server (Campus Social Backend)

![license](https://img.shields.io/badge/license-Apache%202.0-blue)
![java](https://img.shields.io/badge/Java-8-orange)
![springboot](https://img.shields.io/badge/Spring%20Boot-2.7.18-brightgreen)

**Backend service deployment package for a campus social network / campus community system.** Built with Spring Boot 2.7, this repo ships the server JAR, admin web UI, database initialization scripts and a one-click deployment script — ready to run out of the box. Pairs with the [campus mini-program frontend](https://github.com/xiaotuantuankeji/school-circle-mini).

## Features

- RESTful APIs for campus feeds, campus circles, course schedules, playmate matching and more
- Built-in admin dashboard (`schoolAdmin/`) for user/content/verification management
- Database init script (`school_circle.sql`) with full schema & seed data
- One-click deploy script (`deploy.sh`): backup → stop → deploy → start → health check
- Tech stack: Spring Boot 2.7.18 / MySQL / Redis / Nginx

## Requirements

| Dependency | Version | Note |
|---|---|---|
| JDK | 1.8+ | Built with JDK 8 |
| MySQL | 5.7 / 8.0 | Port `3306` |
| Redis | 5.0+ | Port `6379` |
| Nginx | Optional | Hosts the admin page |

## Quick Start

```bash
# 1. Create database and import schema
mysql -uroot -p -e "CREATE DATABASE IF NOT EXISTS school_circle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -uroot -p school_circle < school_circle.sql

# 2. Start Redis with password (default: xtt@2026)

# 3. Run the server (port 48080)
java -jar yudao-server.jar --spring.profiles.active=prod
```

See the [Chinese README](README.md) for the full deployment guide (including how to restore the multi-part JAR archive from `yudao-server.jar.zip` + `yudao-server.jar.z01`).

## Related Repos

- Backend (this repo): [school-circle-server](https://github.com/xiaotuantuankeji/school-circle-server)
- Frontend: [school-circle-mini](https://github.com/xiaotuantuankeji/school-circle-mini)

## License

Licensed under the [Apache License 2.0](LICENSE).

Copyright &copy; 2026 Nanjing Xiaotuantuan Technology Co., Ltd.
