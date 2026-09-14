PORTNAME=	qtpass
DISTVERSIONPREFIX=	v
DISTVERSION=	1.8.0
CATEGORIES=	sysutils

MAINTAINER=	brouwer@annejan.com
COMMENT=	Qt GUI for pass, the standard unix password manager
WWW=		https://qtpass.org/

LICENSE=	GPLv3
LICENSE_FILE=	${WRKSRC}/LICENSE

RUN_DEPENDS=	pass:sysutils/password-store

USES=		compiler:c++17-lang cpe gl qmake:outsource qt:6
USE_GITHUB=	yes
GH_ACCOUNT=	IJHack
GH_PROJECT=	QtPass
USE_GL=		opengl
USE_QT=		base tools:build svg:run translations:run
CPE_VENDOR=	ijhack

# Create a desktop entry for the port.
DESKTOP_ENTRIES=	"QtPass" "" \
			"${PREFIX}/share/pixmaps/qtpass.png" \
			"${PREFIX}/bin/qtpass" \
			"Security;Utility;" true

PLIST_FILES=	bin/qtpass \
		share/pixmaps/qtpass.png

# Install the icon of the port.
post-install:
	${INSTALL_DATA} ${WRKSRC}/artwork/icon.png \
			${STAGEDIR}${PREFIX}/share/pixmaps/qtpass.png

.include <bsd.port.mk>
