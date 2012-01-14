#!/usr/bin/env python

import os
import sys
import warnings

warnings.filterwarnings(
    action='ignore',
    category=RuntimeWarning,
    module='linode.api',
    lineno=59
)

from linode.api import Api

API_KEY = os.environ.get('LINODE_API_KEY')

def list_domains(l, args):
    for domain in l.domain_list():
        print domain['DOMAIN']

def _domain_id_by_name(l, domain_name):
    for domain in l.domain_list():
        if domain['DOMAIN'] == domain_name:
            return domain['DOMAINID']

def _records_for_domain(l, domain_id, record_type=None):
    all_records = l.domain_resource_list(DomainID=domain_id)

    for record in all_records:
        if record_type and record_type != record['TYPE']:
            continue

        yield record

def _add_record(l, domain_id, record_type, **kwargs):
    l.domain_resource_create(
        DomainID=domain_id,
        Type=record_type,
        **kwargs
    )

def _a_record_add(l, domain_id, is_aaaa, args):
    if len(args) != 2:
        print "usage"
        sys.exit(1)

    name, ip = args

    _add_record(
        l,
        domain_id,
        'a' if not is_aaaa else 'aaaa',
        Name=name,
        Target=ip
    )

_a_actions = {
    'add': _a_record_add
}

def _a_record(l, args, is_aaaa=False):
    if len(args) == 0:
        print "usage"
        sys.exit(1)

    domain_name = args[0]
    args = args[1:]

    domain_id = _domain_id_by_name(l, domain_name)
    record_type = 'A' if not is_aaaa else 'AAAA'

    if len(args) == 0:
        # list all A records
        names_and_ips = [
            (record['NAME'], record['TARGET'])
            for record in _records_for_domain(l, domain_id, record_type)
        ]

        max_name_width = min(8, max(len(name) for name, ip in names_and_ips))

        for name, ip in names_and_ips:
            print name + ' ' * (max_name_width - len(name) + 4) + ip

        return

    action = args[0]
    args = args[1:]

    if action not in _a_actions:
        print "usage"
        sys.exit(1)

    _a_actions[action](l, domain_id, is_aaaa, args)

def a_record(l, args):
    return _a_record(l, args, is_aaaa=False)

def aaaa_record(l, args):
    return _a_record(l, args, is_aaaa=True)

_actions = {
    'ls': list_domains,
    'a': a_record,
    'aaaa': aaaa_record
}

if __name__ == '__main__':
    args = sys.argv[1:]

    if len(args) == 0:
        print "usage"
        sys.exit(0)

    l = Api(API_KEY)

    action = args[0]
    args = args[1:]

    if action not in _actions:
        print "blah"
        sys.exit(1)

    _actions[action](l, args)
