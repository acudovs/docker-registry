#!/usr/bin/env python2.7

import argparse
import logging
import re
import requests
import shutil
import urlparse

__author__ = 'Aleksey Chudov <aleksey.chudov@gmail.com>'
__date__ = '21 Jan 2017'
__version__ = '1.0.2'

USAGE = """usage examples:
    docker-registry catalog [name_regex] [tag_regex]
    docker-registry images [name_regex] [tag_regex]
    docker-registry manifest name tag
    docker-registry blob name layer_digest output_file
    docker-registry delete name image_digest
"""


class RegistryObject(object):
    headers = ()

    def __init__(self, *fields):
        for key in range(len(self.headers)):
            try:
                field = fields[key]
            except IndexError:
                field = '<none>'

            setattr(self, self.headers[key], field)

    def __iter__(self):
        for key in range(len(self.headers)):
            yield getattr(self, self.headers[key])

    def __getitem__(self, key):
        return getattr(self, self.headers[key])


class DockerCatalog(RegistryObject):
    headers = ('REGISTRY', 'NAME', 'TAG')


class DockerImage(RegistryObject):
    headers = ('REGISTRY', 'NAME', 'TAG', 'DIGEST', 'SIZE')


class DockerRegistry(object):
    def __init__(self, args):
        self._args = args
        self._registry = urlparse.urlparse(self._args.url).netloc
        self._session = self._create_session()

    def _create_session(self):
        session = requests.Session()
        session.headers = self._args.headers
        session.timeout = self._args.timeout
        session.verify = self._args.verify
        if self._args.username and self._args.password:
            session.auth = (self._args.username, self._args.password)
        return session

    @staticmethod
    def _is_match(pattern, string):
        return not pattern or re.search(pattern, string)

    def _response(self, method, path):
        session_method = getattr(self._session, method)
        response = session_method(self._args.url + path)
        logging.debug('headers:{}'.format(response.headers))
        logging.debug('text:{}'.format(response.text))
        return response

    def get(self, path):
        return self._response('get', path)

    def delete(self, path):
        return self._response('delete', path)


class RegistryCatalog(DockerRegistry):
    _return_class = DockerCatalog

    def action(self, path='/v2/_catalog'):
        response_json = self.get(path).json()

        images = []
        for name in response_json.get('repositories'):
            if self._is_match(self._args.name_regex, name):
                images.extend(self._get_images(name))

        self._args.printer(self._return_class.headers, images).print_out()

    def _get_images(self, name):
        path = '/v2/{}/tags/list'.format(name)
        response_json = self.get(path).json()

        images = []
        if response_json.get('tags'):
            for tag in response_json.get('tags'):
                if self._is_match(self._args.tag_regex, tag):
                    images.append(self._get_image(name, tag))
        else:
            images.append(self._return_class(self._registry, name))

        return images

    def _get_image(self, name, tag):
        return self._return_class(self._registry, name, tag)


class RegistryImages(RegistryCatalog):
    _return_class = DockerImage

    def _get_image(self, name, tag):
        path = '/v2/{}/manifests/{}'.format(name, tag)
        response = self.get(path)
        response_json = response.json()
        image_digest = response.headers.get('docker-content-digest')
        size = self._get_image_size(response_json)
        return DockerImage(self._registry, name, tag, image_digest, size)

    @staticmethod
    def _get_image_size(response_json):
        size = 0
        if response_json.get('schemaVersion') == 2:
            for layer in response_json.get('layers'):
                size += layer['size']
        return size


class RegistryDelete(DockerRegistry):
    def action(self):
        path = '/v2/{}/manifests/{}'.format(
            self._args.name, self._args.image_digest)
        response = self.delete(path)
        if not response.ok:
            logging.error(response.text.strip())


class RegistryManifest(DockerRegistry):
    def action(self):
        path = '/v2/{}/manifests/{}'.format(self._args.name, self._args.tag)
        response = self.get(path)
        print(response.text.strip())


class RegistryBlob(DockerRegistry):
    def action(self):
        path = '/v2/{}/blobs/{}'.format(
            self._args.name, self._args.layer_digest)
        response = self._session.get(self._args.url + path, stream=True)
        logging.debug('headers:{}'.format(response.headers))
        with open(self._args.output_file, 'wb') as output_file:
            shutil.copyfileobj(response.raw, output_file)


class PlainPrinter(object):
    def __init__(self, headers, fields):
        self._headers = headers
        self._fields = fields

    def print_out(self):
        line = ' '.join('{{{}}}'.format(i) for i in range(len(self._headers)))
        for field in self._fields:
            print(line.format(*field))


class TablePrinter(object):
    def __init__(self, headers, fields):
        self._headers = headers
        self._fields = fields
        self._total_line = 'Total: {}'
        self._header_lens = self._get_header_lens()
        self._cross_line = self._get_cross_line()

    def _get_header_lens(self):
        header_lens = [len(str(header)) for header in self._headers]
        total_len = len(self._total_line.format(len(self._fields)))

        if sum(header_lens) < total_len:
            header_lens[-1] = total_len

        for field in self._fields:
            for i in range(len(header_lens)):
                field_len = len(str(field[i]))
                if header_lens[i] < field_len:
                    header_lens[i] = field_len

        return header_lens

    def _get_cross_line(self):
        return '+-{}-+'.format('-+-'.join('-' * i for i in self._header_lens))

    def _print_fields(self, fields):
        sub_fields = []
        for header, header_len in zip(fields, self._header_lens):
            sub_fields.append('{:<{}}'.format(header, header_len))
        print('| {} |'.format(' | '.join(sub_fields)))

    def _print_total(self):
        total_len = sum(self._header_lens) + (len(self._headers) - 1) * 3
        total_format = (self._total_line.format(len(self._fields)), total_len)
        print('| {:>{}} |'.format(*total_format))

    def print_out(self):
        print(self._cross_line)
        self._print_fields(self._headers)
        print(self._cross_line)

        for field in self._fields:
            self._print_fields(field)

        print(self._cross_line)
        self._print_total()
        print(self._cross_line)


class DockerImages(object):
    DEFAULT = {
        'url': 'http://localhost:5000',
        'timeout': 3,
        'username': '',
        'password': '',
        'headers': {
            'accept': 'application/vnd.docker.distribution.manifest.v2+json'
        }
    }

    def __init__(self):
        self._args = self._parse_args()
        self._logging_config()
        try:
            self._args.executor_class(self._args).action()
        except Exception as e:
            if self._args.level == 'DEBUG':
                raise
            print(e)

    def _logging_config(self):
        logging.basicConfig(level=self._args.level)
        logging.debug('args:{}'.format(self._args))

    def _parse_args(self):
        parser = argparse.ArgumentParser(
            formatter_class=argparse.RawTextHelpFormatter, epilog=USAGE)

        parser.add_argument('-d', '--debug', action='store_const',
                            const='DEBUG', default='WARNING', dest='level')

        parser.add_argument('-f', '--plain', action='store_const',
                            const=PlainPrinter, default=TablePrinter,
                            dest='printer')

        parser.add_argument('-l', '--url')
        parser.add_argument('-t', '--timeout')
        parser.add_argument('-u', '--username')
        parser.add_argument('-p', '--password')
        parser.add_argument('-k', '--insecure', action='store_false',
                            dest='verify')

        subparsers = parser.add_subparsers(dest='action')

        subparser = subparsers.add_parser('catalog')
        subparser.add_argument('name_regex', nargs='?')
        subparser.add_argument('tag_regex', nargs='?')
        subparser.set_defaults(executor_class=RegistryCatalog)

        subparser = subparsers.add_parser('images')
        subparser.add_argument('name_regex', nargs='?')
        subparser.add_argument('tag_regex', nargs='?')
        subparser.set_defaults(executor_class=RegistryImages)

        subparser = subparsers.add_parser('delete')
        subparser.add_argument('name')
        subparser.add_argument('image_digest')
        subparser.set_defaults(executor_class=RegistryDelete)

        subparser = subparsers.add_parser('manifest')
        subparser.add_argument('name')
        subparser.add_argument('tag')
        subparser.set_defaults(executor_class=RegistryManifest)

        subparser = subparsers.add_parser('blob')
        subparser.add_argument('name')
        subparser.add_argument('layer_digest')
        subparser.add_argument('output_file')
        subparser.set_defaults(executor_class=RegistryBlob)

        return parser.parse_args(namespace=argparse.Namespace(**self.DEFAULT))


if __name__ == '__main__':
    DockerImages()
